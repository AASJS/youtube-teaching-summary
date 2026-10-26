# 运行日志（runlog）

> 每次运行本 skill 后**必须**在「日志」表末尾追加一行；每次运行前**必须**先通读本表，规避已知失败模式。
> 这是本 skill 迭代的原料：同类问题出现 2–3 次 → 升版修复（见仓库 ROADMAP.md）。

## 日志格式

```
| 日期 | 视频ID | 类型 | noembed命中 | 主路径(WebSearch)命中 | 用了几篇来源 | 产出文件 | 用户反馈 | 备注/坑 |
|------|--------|------|------------|----------------------|-------------|----------|----------|---------|
```

字段说明：
- **类型**：long（普通视频）/ shorts / playlist
- **主路径命中**：WebSearch 是否找到 ≥2 篇引用该视频的来源（是/否/部分）
- **用户反馈**：满意 / 未反馈 / 不满意（+原因，如「章节对不上」「太浅」）
- **备注/坑**：本次遇到的任何意外，例如 noembed 对 Shorts 失效、某字幕站短暂可用等

## 已知经验（从历史运行沉淀，运行前先看）

- **Shorts 链接必须先规范化**：`/shorts/<ID>` 转 `watch?v=<ID>` 再喂 noembed，否则失效（v1.1.0 已写死）。
- **noembed 连续快速调用会限流**：批量验证多个 ID 时，第 6 个请求开始超时/空返回，sleep 5s 重试即恢复 → 批量场景每请求间隔 3–5s。
- **部分视频 noembed 返回 403**（如李永乐 T93SayXhw2w）：视频不在 noembed 收录内 → 用 v1.1.0 兜底：WebSearch 直接搜视频 ID，实测有效。
- **禁止凭频道猜内容**：Mosh 的 Ke90Tje7VS0 被误当 Python 教程搜索（命中全是频道其他视频），拿到 noembed 标题后才发现是 React 教程 → **搜索关键词必须用精确标题，不能凭频道推断主题**。
- **同名多版本甄别**：Traversy《React JS Crash Course》有 2021(w7ejDZ8SWv8) 与 2024(3h5m新版) 两个版本，章节表完全不同 → 优先用「按视频 ID 建页」的来源（如 `glasp.co/youtube/<ID>`），并用视频时长/发布年份交叉核对章节表归属。
- **重大发现：自动字幕站是字幕直连的替代品**：rosetta.to / recapio.com / glasp.co / sozai.app 这类站会自动从 captions 生成「章节 + 时间戳 + 摘要」页面（如 rosetta.to/u/fireship/<slug>、glasp.co/youtube/<ID>、sozai.app/transcript/<slug>）。热门英文视频在这类站命中率极高，等于**绕过了字幕直连被墙的问题** → 搜索时优先加上 `site:rosetta.to / site:glasp.co / site:recapio.com / site:sozai.app` 组合。
- **B 站元数据路径已验证（v1.2.0 素材）**：noembed 不支持 B 站，但官方 API `https://api.bilibili.com/x/web-interface/view?bvid=<BV号>` 免鉴权可达，返回 title/owner 完整 → v1.2.0 的 B 站支持直接用此 API 替代 noembed。
- **自动字幕站不可 curl 直连（v1.2.0 实测补充）**：glasp.co SSL 握手被拦、rosetta.to 403（bot 拦截）、recapio.com 200 → v1.2.0 设计正确：**靠 WebSearch 索引命中这些站的内容摘要，不要 curl/WebFetch 直连**。
- **B 站多 P 合集目录免费拿（v1.3.0 黄金素材）**：view API 的 `data.pages` 数组返回全部分 P（P 号/标题/cid/时长），实测狂神说 Java（BV12J41137hu）返回 80 个分 P 完整目录 → **v1.3.0「系列视频支持」在 B 站侧有现成结构，比 YouTube 播放列表容易**，建议 B 站先做。
- **YouTube 播放列表 noembed 不支持（v1.3.0 目标场景实证）**：`playlist?list=...` 返回 `no matching providers found` → 播放列表需专门处理路径；单视频 URL 带 `&list=` 参数时按现有规则剥掉即可不受影响。
- **B 站官方字幕/摘要接口存在（v1.3.0 探索方向）**：BiliTools / Bili Note 两个开源工具实证 B 站有「官方 AI 摘要」与「公开字幕」接口（view API 返回的 aid/cid 是取字幕的钥匙）→ v1.3.0 可探索，若走通，B 站内容路径将升级为「准字幕级」。

## 日志

- 2026-10-26 | FKXRiAiQFiY | long | ✅ | ✅ | 3 | Git_GitHub视频教学内容总结.md | 满意 | PAPAYA Git 入门；WebSearch 命中官方简介+iT邦笔记+technice
- 2026-10-26 | 3VD0-qerrOI | shorts | ❌（/shorts/ 不识别）→ 规范化后 ✅ | ✅ | 6+ | YouTube教学内容总结_Hypit_AI克隆视频.md | 满意 | 首个 Shorts；引出规范化规则
- 2026-10-26 | bWUUHBVg-7E | long | ✅ | ✅ | 2 | YouTube教学内容总结_技术爬爬虾_Git_GitHub核心概念.md | 满意 | 微信公众号笔记+CSDN 笔记两源一致
- 2026-10-26 | aircAruvnKk | long | ✅ | ✅ | 5 | （批量 runlog run，见 docs/batch-run-2026-10-26.md） | 未反馈 | 3Blue1Brown 神经网络；官方 lesson 页 + sozai.app 带时间戳 transcript + glasp/podcore，来源生态极丰富
- 2026-10-26 | kCc8FmEb1nY | long | ✅ | ✅ | 3 | 同上 | 未反馈 | Karpathy GPT；idvorkin「31 章节 mapped」+ 掘金 13h 课指南，中文笔记覆盖好
- 2026-10-26 | -uleG_Vecis | long | ✅ | ✅ | 4 | 同上 | 未反馈 | Fireship 100+ CS；rosetta/recapio 自动字幕站带完整时间戳，glasp 按 ID 建页
- 2026-10-26 | w7ejDZ8SWv8 | long | ✅ | ✅ | 3 | 同上 | 未反馈 | Traversy React 2021；官方完整时间戳被 cyberbump/piratekingdom 原样收录；**同名 2024 版干扰需甄别**
- 2026-10-26 | nykOeWgQcHM | long | ✅ | ✅ | 4 | 同上 | 未反馈 | MIT 6.0001 L1；OCW 官方 PDF 讲义可达 + 两篇 CSDN 中文笔记
- 2026-10-26 | Ke90Tje7VS0 | long | ⚠️ 首次限流失败，sleep 5s 重试成功 | ⚠️ 首搜凭频道猜错主题，按精确标题重搜后 ✅ | 3 | 同上 | 未反馈 | Mosh React 教程；双教训：限流 + 禁止凭频道猜内容
- 2026-10-26 | T93SayXhw2w | long | ❌ 403 Forbidden → 兜底 WebSearch ID ✅ | ✅ | 2 | 同上 | 未反馈 | 李永乐黎曼猜想；v1.1.0 兜底路径首次实战验证通过；中文科普类来源以博客+公众号为主
- 2026-10-26 | BV12J41137hu | long（多P合集） | 官方 API ✅（B站分支首跑） | ✅ | 2+ | （B站分支实测，未出全量总结） | 未反馈 | 狂神说 Java 零基础（17.2h/80 分P）；view API 全字段返回 + pages 目录完整；CSDN 逐节笔记明确按 BV 号命中；v1.2.0 B站分支验证通过
- 2026-10-26 | PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi | playlist | ❌ no matching providers | n/a | 0 | （播放列表探测，未产出） | 未反馈 | YouTube 播放列表 noembed 不支持 → v1.3.0 需专门路径；watch+list 单视频场景不受影响（list 参数剥离规则已覆盖）
