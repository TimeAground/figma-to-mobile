# 问题清单 — figma-to-mobile(2026-08-06)

> 评测框架:Skill Standard Evaluator V2.0 | 首次评测
> 优先级:P0(致命,触发一票否决)/ P1(严重,强制修复)/ P2(一般)/ P3(提示)

| ID | 所属维度 | 优先级 | 问题描述 | 证据 | 改进建议 |
|----|---------|:------:|---------|------|---------|
| ISS-01 | D6 可维护性 | **P0** | frontmatter 无 `version` 字段、无 CHANGELOG,版本管理确定性检查 0% | SKILL.md frontmatter 仅 name/description/metadata | 添加 `version: 1.0.0`(语义化版本),新增 CHANGELOG.md |
| ISS-02 | D6 可维护性 | **P0** | 无 `eval/evals.json` 自测用例集,自测覆盖确定性检查 0% | 项目无 eval/ 目录 | 新增 evals.json,含 P0 用例 ≥ 10 条(链接类型判断/错误处理/多帧流程) |
| ISS-03 | D1 功能正确性(结构性) | **P1** | 缺标准 `requires` 字段(C1,High 降级) | frontmatter 仅 metadata.openclaw.requires | 添加 `requires: [Read, Write, Grep, Glob, RunCommand]` |
| ISS-04 | D1 功能正确性(结构性) | **P1** | 缺独立交付物章节(C5,High 降级) | SKILL.md 无 Output/Deliverables 章节 | 新增 `## 交付物` 章节,列出代码文件/scan-report.json/feedback-log.md |
| ISS-05 | D4 鲁棒性安全(结构性) | **P1** | 缺独立不适用场景章节(C7,High 降级) | 不触发说明仅内嵌于 Trigger & Input | 新增 `## 不适用场景` 章节(截图转码、纯聊天、无链接等) |
| ISS-06 | D5 体验对齐 | P2 | description 约 480 字符,超 200 上限(C3) | frontmatter description 多行折叠 | 精简至 200 字符内,细节移入正文 |
| ISS-07 | D3 效率成本 | P2 | SKILL.md ≈ 3,338 tokens,命中 AP-001 万能 Skill | 270 行 / 12,756 字符 | 将低频示例(多帧对话模板)下沉至 references/ |
| ISS-08 | D6 可维护性 | P2 | 目录含 `__pycache__/` 缓存(C11) | scripts/、scripts/scanners/、tests/ 下 3 个缓存目录 | 删除缓存目录,加入 .gitignore |
| ISS-09 | D4 鲁棒性安全 | P2 | 缺 resource_manifest 标准声明(R1) | 仅有 metadata.permissions | 添加 resource_manifest(cpu/memory/timeout/network) |
| ISS-10 | D5 体验对齐 | P3 | 无 Tips/经验章节(C10) | SKILL.md 无相关章节 | 补充 3 条最佳实践(如 token 安全管理、多帧限速经验) |
| ISS-11 | D6 可维护性 | P3 | 无版本号/变更日志,命中 AP-009 版本游离 | frontmatter 无 version | 同 ISS-01 |
| ISS-12 | D3 效率成本 | P3 | 网络请求无超时/重试显式声明 | SKILL.md 有限速但无超时 | 声明 API 请求超时与重试上限 |

**严重度汇总**:Critical 0 项 | High 3 项(ISS-03/04/05)→ 降一级处置 | Medium 4 项 | Low 2 项

**P0 修复后可解锁评级**:ISS-01 + ISS-02 修复后,D6 确定性 0→100,D6 维度分 20→80,跨过乘法闸门;ISS-03/04/05 修复后消除 High 降级。
