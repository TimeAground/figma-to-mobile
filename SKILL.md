---
name: figma-to-mobile
description: >
  Convert Figma designs to mobile UI code.
  Supports Android (Jetpack Compose, XML) and iOS (SwiftUI, UIKit).
  Use when a user provides a Figma link and wants mobile layout code.
  Extracts design tokens via Figma REST API, asks clarifying questions,
  then generates production-ready code files.
  Bugs/feedback: https://github.com/TimeAground/figma-to-mobile/issues
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["python3"], "env": ["FIGMA_TOKEN"] },
        "primaryEnv": "FIGMA_TOKEN",
        "install":
          [
            {
              "id": "python-requests",
              "kind": "shell",
              "command": "pip3 install requests",
              "label": "Install Python requests package",
            },
          ],
      },
  }
---

# Figma to Mobile

Convert Figma designs to mobile UI code with interactive clarification.

Supported: Android Compose, Android XML, iOS SwiftUI, iOS UIKit.

## Prerequisites

- `FIGMA_TOKEN` environment variable set (Figma > Settings > Personal Access Tokens)
  ⚠️ **Your Figma token is sensitive** — treat it like a password. Never paste it into chat
    messages (they may be logged). Set it via your shell rc file or OpenClaw env config.
- Python 3.8+ with `requests` package

## Trigger & Input

This skill activates when a user provides a Figma link.

The user may also include **inline hints** alongside the link, such as:
- Target platform: "Android XML", "Compose", "SwiftUI", "UIKit"
- Layout preferences: "use ConstraintLayout", "prefer StackView"
- Component notes: "the switch is our custom CompactSwitch", "this is a dynamic list"
- Any other context about the design

**If the user provides hints, respect them and skip the corresponding questions.**
For example, if the user says "Android XML, the 3 cards are a RecyclerView list", do NOT ask about output format or whether the cards are dynamic/static.

## Workflow

### Step 1: Fetch & Analyze

When user provides Figma link(s):

1. **Determine the input type:**

   **A. Link without specific node-id** (no `node-id`, or `node-id=0-1`):
   This link points to the entire page, not a specific frame. Tell the user:
   > This link points to the whole page. Please select the frame you want in Figma, right-click it, and choose "Copy link to selection", then send that link.
   > If you want to convert multiple frames, send multiple links.

   **B. Single frame link** (has specific `node-id`):
   Run `scripts/figma_fetch.py "<url>"` → returns that frame's design data.
   Proceed to analysis.

   **C. Multiple links** (user sends 2+ URLs):
   First, determine the relationship by examining frame names and user context:

   - **Same page, different visual states** (e.g. "首页-有banner" and "首页-无banner"): Use `--compare` mode to fetch all and get a diff summary. Generate multi-state code (conditional visibility, state switching).

   - **Parent page + overlay/drawer** (e.g. "首页" + "首页-抽屉-xxx"): Generate each as an **independent layout file**. Then tell the user the relationship:
     > Frame 1 ("首页") and Frame 2 ("首页-抽屉") look like a main page + side drawer.
     > I've generated two separate layout files. How you wire them together (DrawerLayout, Navigation, etc.) depends on your project architecture.

     The Skill's job is generating UI layout code, not deciding architecture (Activity vs Fragment vs Navigation).

   - **Different independent pages** (e.g. "首页" + "设置页" + "个人中心"): Process each independently. Fetch them **one at a time** with a pause between requests to avoid rate limiting. Present a summary of all pages, then ask which to convert first (or convert all sequentially).

   - **Not sure**: Ask the user — "These frames look related but I'm not sure how. Are they different states of the same page, a page with an overlay, or independent pages?"

   **Rate limit protection for multiple links**: When fetching multiple nodes, wait 2-3 seconds between requests. Never fire more than 2 requests in parallel.

2. **If the link has no specific node-id**, ask the user to re-copy from the specific frame (see A above). Do NOT call the API.

3. Analyze the structure: identify sections, repeated patterns, component types
4. Note INSTANCE nodes — they indicate reusable components. Check `variantProperties` for component state (e.g. State=Default, Size=Large) — these map to multi-state code
5. Note gradient/shadow data — flag for the user if complex
6. Apply Figma node interpretation rules before generating code

**Detailed interpretation rules**: Read `references/figma-interpretation.md`

### Step 1.5: Structure Summary

Before asking any questions, present a brief **structure summary** to the user so they can confirm your understanding:

> I see: [navigation bar with back button + title] → [2 content sections: user profile card, settings list (8 items)] → [bottom action button]. Total ~25 nodes.

Keep it to 2-3 lines. Mention:
- Major sections identified (nav bar, content areas, footer)
- Repeated patterns ("8 similar list items", "3 tab labels")
- Notable elements (gradients, complex illustrations, stacked cards)

If the user says "that's wrong" or corrects the structure, adjust understanding before proceeding to Step 2.

If the design has ≤10 leaf nodes (visible elements that map to actual views), skip this step — the structure is simple enough to proceed directly.

### Step 2: Confirm & Clarify

**Question priority (strict order — ask earlier questions first):**

1. **Output format** (MUST ask first unless user already specified)
   → Android XML / Compose / SwiftUI / UIKit
   This determines all subsequent analysis phrasing and code output.

2. **Structural ambiguities** (only ask what you're genuinely unsure about)
   → "These N items look similar — dynamic list or fixed layout?"
   → "This area: single image asset or icon-on-background combo?"

3. **Component choices** (only if platform-relevant)
   → "Any custom components to use? (otherwise I'll use platform defaults)"

**Rules for questions:**
- Skip any question the user already answered via inline hints
- Max 3-5 questions total, fewer is better
- Each question gives concrete options with one-line pros/cons
- Every question includes an open option: "or tell me more about this"
- Use natural language, no JSON or technical dumps
- If everything is clear (user gave full context + simple structure), skip Step 2 entirely

**Confidence guide — when to ask vs. when to just generate:**
- ≥3 sibling nodes with similar structure → likely a list → ASK (dynamic vs static)
- INSTANCE nodes sharing same componentId → reusable component → MENTION but can default
- Single clear hierarchy, no ambiguity → high confidence → SKIP questions, go to Step 3
- Gradient/complex shadow in design → MENTION in summary ("I see a gradient here, I'll approximate it as X")

### Step 2.5: Project Scan — Ask First

**⚠️ Always ask the user before scanning their project.** Scanning reads local files;
the user should know and agree.

> "你的项目在 /path/to/project 对吗？要不要我先扫描一下项目里已有的资源
>（颜色、文案、图片、自定义组件），这样生成代码时可以直接复用已有的东西？"

If the user agrees:

```bash
python scripts/project_scan.py /path/to/project --json --output scan-report.json
```

Then read `scan-report.json` and `references/scan-usage.md`.

If the user declines: proceed with hardcoded values per generation rules.

**How to present scan results:**

Keep it brief and useful — not a JSON dump:

> ✓ 扫描完成：找到 3 个模块、24 个颜色、18 条文案。
> 其中有 6 个颜色映射到了主题色（primary、surface 等），生成代码时会用项目资源引用。

**If scan found issues, tell the user naturally:**

> 扫描完了，找到 N 个资源。不过 [具体问题，如某个模块没找到资源文件]，
> 你项目里这部分是怎么组织的？

**If no project path is known yet, don't scan.** Proceed with hardcoded generation.

### Step 3: Generate Code

After user confirms (or if no questions needed), generate code files.

**Detailed generation rules**: Read `references/generation-rules.md`

If multiple files are needed, output each with a clear filename header:
```
📄 activity_notification_settings.xml
[code]

📄 item_expert_notification.xml
[code]
```

### Step 4: Iterate & Capture Feedback

After showing code, ask briefly:
> Matches the design? Any adjustments?

**The user can then give feedback to refine the output.** Common iterations:
- "间距大了" → adjust specific spacing
- "Switch 换成我们的 CustomSwitch" → swap component
- "把标题栏去掉" → remove section
- "换成 Compose 版本" → regenerate in different format
- "颜色不对，这里应该是 #333333" → fix specific values

Continue iterating until the user is satisfied.

**Iteration output rule:**
- If the file has already been written to disk → read the current file, apply only the minimal patch, output just the changed lines with clear context (file path + line range). Do NOT regenerate the whole file.
- If the code only exists in the conversation (not written to disk) → output only the changed snippet with a comment indicating where it replaces (e.g., `// replaces lines 12-18 in activity_main.xml`). Do NOT repeat the entire file.
- Only regenerate the full file if the user explicitly asks (e.g., "重新生成完整文件", "show me the full file").

**⚠️ Before logging any feedback, tell the user:**
  > "I'll save this correction locally to `feedback-log.md` to improve future output.
  > It stores before/after snippets — is that OK?"
  Only proceed if the user agrees.

**⚠️ IMPORTANT: Every time the user corrects your output (layout issue, wrong component, spacing problem, etc.), you MUST log it to `feedback-log.md` before proceeding with the fix (after user consent). Do not skip this step — the log is how the skill learns and improves over time.**

**Feedback capture:**
Whenever the user corrects your generated output (with consent), log the correction to `feedback-log.md` in the project root (create if it doesn't exist). Each entry follows this format:

```
## YYYY-MM-DD HH:MM
- **Platform**: Android XML / Compose / SwiftUI / UIKit
- **Figma node type**: (e.g., FRAME with icon, Tab bar, Button group)
- **Issue**: Brief description of what was wrong
- **Before**: What the agent generated (snippet or description)
- **After**: What the user wanted (snippet or description)
- **Rule candidate**: (optional) If this correction suggests a general pattern rule, note it here
```

Log entries should be:
- **Concise** — only the relevant diff, not entire files
- **Categorized** — always include platform and Figma node type for later analysis
- **Actionable** — focus on the mapping error, not cosmetic preferences (e.g., "user prefers 16dp" is not a rule; "VECTOR compositions should be single ImageView" is)

Do NOT log:
- One-off personal preferences (specific color choices, naming conventions)
- Corrections to non-mapping issues (typos, import statements)
- Feedback the user explicitly says is project-specific, not general

Periodically (or when asked), run `scripts/feedback_analyze.py` to identify patterns and generate rule candidates.

## Error Handling

- **FIGMA_TOKEN not set** (script outputs `FIGMA_TOKEN_NOT_SET`):
  Tell the user:
  > I need a Figma Personal Access Token to fetch the design.
  > ⚠️ **Do not paste it into this chat** — chat messages may be logged.
  > Set it as an environment variable and restart.
  > Get one at: Figma → avatar → Settings → Security → Personal Access Tokens
  >
  >   Windows: `setx FIGMA_TOKEN "figd_xxx"`
  >   macOS/Linux: add `export FIGMA_TOKEN="figd_xxx"` to ~/.zshrc
- **FIGMA_TOKEN invalid** (API returns 403/401) → tell user the token may have expired or been revoked.
  Direct them to regenerate from Figma Settings → Security → Personal Access Tokens.
- **Invalid URL** → show valid URL example: `https://www.figma.com/design/<fileKey>/<name>?node-id=<id>`
- **API error** → show error message, suggest checking network/proxy
- **Node too large (>200 children)** → suggest selecting a smaller frame
- **Depth auto-increased** → the script auto-retries with deeper depth if it detects truncated children. Inform user if this happens ("I needed to fetch deeper to get all details").
