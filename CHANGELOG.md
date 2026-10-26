# Changelog

All notable changes to this skill will be documented in this file.

## [1.1.0] - 2026

### Added
- **运行日志机制（runlog）**：新增 `references/runlog.md`，运行前必读「已知经验」规避坑，运行后必写一条日志；同类问题 2–3 次即触发升版修复（SKILL.md 新增 Step 0 / Step 7，workflow.md 新增 F 节）
- **Shorts / 标题获取兜底（写死）**：Shorts 链接强制先规范化为 `watch?v=<ID>` 再调 noembed；noembed 与 oembed 双失败时，兜底用 WebSearch 直接搜视频 ID/URL 拿标题（决策树已更新）
- 3 条历史运行记录入库（PAPAYA Git 入门 / Hypit Shorts / 爬爬虾 Git 概念）

## [1.0.1] - 2026

### Changed
- `display_name` 改为「YouTube 教学视频总结助手」（原名称在平台被占用）
- `display_name_en` 改为「YouTube Tutorial Summary Assistant」

### Fixed
- 补齐平台要求 frontmatter 字段：`version` / `display_name` / `display_name_en` / `description_zh` / `description_en`
- 打包格式改为标准正斜杠 zip（bsdtar），修复跨平台解压问题

## [1.0.0] - 2026

### Added
- 初始版本：粘贴 YouTube 教学/知识视频链接 → 结构化学习笔记
- 核心工作流：noembed 取元数据 → WebSearch 找章节表与第三方笔记 → 多源交叉核对 → 模板化输出
- 受限网络降级链（实测端点可达性清单见 `references/workflow.md`）
- 输出模板 `references/summary_template.md`（分章节 + 速查表 + 来源说明）
- 实测验证：3 个视频（PAPAYA Git 入门 15min / 技术爬爬虾 Git 概念 42min / Hypit Shorts 60s）
