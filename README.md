# figma-to-mobile

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.8+](https://img.shields.io/badge/Python-3.8+-green.svg)](https://python.org)
[![Platforms](https://img.shields.io/badge/Platforms-Android%20%7C%20iOS-orange.svg)](#supported-platforms)

> Convert Figma designs to production-ready mobile UI code using AI.

**Jetpack Compose** · **Android XML** · **SwiftUI** · **UIKit**

## ✨ What It Does

Paste a Figma design link → get idiomatic mobile UI code. Not pixel-positioned boxes — real layouts using `LazyColumn`, `Row`, `VStack`, `UITableView`, etc.

![Figma to Compose comparison](assets/demo-comparison.png)
*Left: Figma design · Right: Generated Jetpack Compose code running in Android Studio*

## 🧠 How It Works

1. **Fetch** — Python script calls Figma REST API to extract the full node tree (auto-layout, style refs, component variants)
2. **Interpret** — AI analyzes layout semantics: "6 similar rows → `LazyColumn`", "horizontal stack with spacing → `Row`"
3. **Generate** — Outputs platform-idiomatic code with proper theming (Material3, SF Symbols, dynamic type)
4. **Iterate** — Refine through natural language: *"make the header sticky"*, *"switch to dark theme"*

## 🔑 Key Differentiators

| | Screenshot-based tools | figma-to-mobile |
|---|---|---|
| **Input** | Screenshot / image | Figma API (design tree) |
| **Layout** | Pixel positions | Auto-layout semantics |
| **Output** | Absolute positioning | Idiomatic code |
| **Iteration** | Re-screenshot | Natural language |
| **Cost** | Paid subscription | Free & open source |

## 🚀 Quick Start

### 1. Setup Figma Token

```bash
# Get token: Figma → Settings → Security → Personal Access Tokens
export FIGMA_TOKEN="figd_your_token_here"
```

### 2. Install

Works with any AI coding assistant that supports agent skills:

```bash
# OpenClaw
clawhub install figma-to-mobile

# Claude Code — copy to your project
cp -r figma-to-mobile/ your-project/.claude/skills/

# GitHub Copilot — copy to your project
cp -r figma-to-mobile/ your-project/.agents/skills/
```

### 3. Use

```
Convert this to Jetpack Compose:
https://www.figma.com/design/xxx/Project?node-id=100-200
```

The AI agent will fetch the design, ask clarifying questions, and generate production-ready code files.

## Supported Platforms

| Platform | Framework | Key Features |
|---|---|---|
| **Android** | Jetpack Compose | Material3, LazyColumn/Row, Navigation, ViewModel-ready |
| **Android** | XML | ConstraintLayout, RecyclerView, DataBinding-ready |
| **iOS** | SwiftUI | SF Symbols, NavigationStack, @Observable |
| **iOS** | UIKit | Auto Layout, UICollectionView, programmatic UI |

## 🏗 Architecture

```
figma-to-mobile/
├── SKILL.md                    # AI agent instructions
├── scripts/
│   ├── figma_fetch.py          # Figma API data fetcher
│   └── project_scan.py         # Existing project scanner
│   └── scanners/               # Platform-specific analyzers
├── references/
│   ├── figma-interpretation.md # Design node → UI mapping rules
│   ├── compose-patterns.md     # Jetpack Compose patterns
│   ├── xml-patterns.md         # Android XML patterns
│   ├── swiftui-patterns.md     # SwiftUI patterns
│   └── uikit-patterns.md      # UIKit patterns
└── tests/
    └── test_project_scan.py    # Scanner unit tests
```

## Requirements

- Python 3.8+ with `requests`
- Figma Personal Access Token (free)
- An AI coding assistant (OpenClaw, Claude Code, GitHub Copilot, etc.)

## License

MIT
