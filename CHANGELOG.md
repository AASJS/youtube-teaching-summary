# Changelog

All notable changes to this skill will be documented in this file.

## [1.3.0] - 2026

### Added
- **系列视频模式**：B 站多 P 合集——view API 的 `pages` 数组即全量目录（P号/标题/cid/时长），逐讲总结 + 汇总「学习路径总表」（P号｜标题｜时长｜核心主题｜依赖前置｜一句话收获）；分 P >10 时先出总表并与用户对齐范围，勿擅自全量跑
- **YouTube 播放列表处理策略（写死）**：实证 noembed 对 `playlist?list=` 返回 `no matching providers` → 改走 WebSearch 找第三方整理的列表；单视频带 `&list=` 参数不受影响（规范化规则已覆盖）
- **输出格式可配置**：默认 Markdown 笔记；可选「小红书图文版」（钩子 + 知识点卡片 + 话题标签）与「公众号文章版」（导语 + 分节 + 参考阅读）
- **B 站字幕级探测（best-effort）**：`api.bilibili.com/x/player/wbi/v2?bvid=&cid=` 尝试拿 `data.subtitle.subtitles`，有则取 `subtitle_url` 下载 JSON 字幕升级为最高保真路径，空/报错立即跳过

### Evidence
- 基于 2026-10-26 v1.3.0 喂养轮实测：B 站分支首跑通过（狂神说 Java BV12J41137hu，80 分 P 目录完整返回）；YouTube 播放列表 noembed 不支持已实证；自动字幕站直连边界确认（rosetta 403 / glasp SSL 拦截 / recapio 200 → 佐证靠 WebSearch 索引命中的设计）

## [1.2.0] - 2026

### Added
- **B 站支持**：识别 `bilibili.com/video/<BV号>` 与 `b23.tv` 短链，元数据走官方 API `api.bilibili.com/x/web-interface/view`（免鉴权，实测可达）；noembed 对 B 站无效已写入端点清单
- **自动字幕站升级为主路径首选**：rosetta.to / glasp.co（按 ID 建页）/ recapio.com / sozai.app——自动从 captions 生成章节+时间戳，绕过字幕直连被墙
- **同名多版本甄别规则（写死）**：核对来源的视频 ID/时长/发布年份，对不上即丢弃（源自 Traversy React 2021/2024 两版实测教训）

### Fixed
- **限流防御写死**：noembed 批量调用每请求间隔 3–5s，失败 sleep 5s 重试一次再走兜底（实测连发第 6 个请求起限流）
- **精确标题铁律**：搜索关键词必须用 Step 1 拿到的精确标题，严禁凭频道名推断主题（Mosh React 被误当 Python 搜的实测教训）

### Evidence
- 基于 2026-10-26 批量实战：7 个视频 100% 产出（详见 `docs/batch-run-2026-10-26.md`）

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
