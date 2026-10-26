---
name: youtube-teaching-summary
version: 1.1.0
display_name: YouTube 教学视频总结助手
display_name_en: YouTube Tutorial Summary Assistant
description: 输入一个 YouTube 知识/教学/课程类视频链接，自动生成结构化的「学习视频教学内容总结」（分章节讲解 + 核心指令/概念速查表）。当直连 YouTube 或字幕代理受限时，自动改用「网页搜索 + 多源交叉核对」的方式可靠产出。触发词：youtube 总结、视频教学内容总结、教学视频笔记、根据视频生成总结、复制一个 youtube 链接生成总结。
description_zh: 粘贴 YouTube 教学/知识类视频链接，自动生成结构化学习笔记：分章节讲解、要点表格、核心指令/概念速查表；网络受限时自动降级为网页搜索多源交叉核对。
description_en: Paste a YouTube tutorial/educational video link and auto-generate a structured study note with chapter breakdowns, key-point tables, and a command/concept cheat sheet; falls back to web search with multi-source cross-checking when direct access is restricted.
---

# YouTube 教学视频内容总结

把一个 YouTube 知识讲解视频，变成一份带章节、带要点、带速查表的结构化学习笔记。

## 何时使用
- 用户给了一个 YouTube 视频链接，要求「总结 / 整理教学内容 / 做笔记 / 提取知识点」。
- 视频类型是：教程、技术讲解、公开课、软件教学、概念科普等「以传授知识为主」的内容。
- 用户想把看过的教学视频沉淀成可复习、可复用的文档。

## 核心工作流（按此顺序，别跳步）

### 0. 读运行日志（每次必做）
打开 `references/runlog.md` 通读「已知经验」与最近几条日志，规避已踩过的坑（如 Shorts 链接需先规范化）。**运行结束后回到第 7 步追加一条日志。**

### 1. 取元数据（几乎必定成功）
**先规范化 URL**：Shorts 链接（`youtube.com/shorts/<ID>`）必须转成 `youtube.com/watch?v=<ID>` 再喂给下面两个端点——noembed 不识别 shorts 形式。
```bash
curl -sS -m 20 "https://noembed.com/embed?url=<规范化后的视频URL>"        # 返回 title / author_name / provider
curl -sS -m 20 "https://www.youtube.com/oembed?url=<规范化后的视频URL>&format=json"  # 备选，可能 401
```
拿到标题（title）、频道（author_name）。**标题是后面搜索的关键词**，必须先拿到。

**兜底（noembed 与 oembed 都失败时）**：直接用 WebSearch 搜视频 URL 或视频 ID（如 `"3VD0-qerrOI"` 或完整链接），从搜索结果的标题/摘要里拿视频标题与频道名——实测有效，不要在此卡死。

### 2. 抓内容（主路径 = 网页搜索，不要死磕直连字幕）
在受限网络里，直连 YouTube 与多数字幕代理经常失败。最稳的是**用标题去搜现成的章节表与学习笔记**：

用 WebSearch，按下面几类查询（中英文都试，语言跟随视频）：
- `"<视频标题>" 章节` / `"<视频标题>" 教學 筆記` / `"<视频标题>" 完整筆記`
- `"<视频标题>" summary` / `"<视频标题>" notes` / `"<视频标题>" chapters`
- `"<视频标题>" site:ithelp.ithome.com.tw` / `site:technice.com.tw`（中文技术笔记常见站）
- 加上视频 ID 或频道名一起搜，提高命中

用 WebFetch 抓 2–3 篇**明确引用了该视频（含视频链接/ID）**的来源，优先：官方视频简介里的章节表、同一作者的配套文章、或第三方学习笔记（如 iT 邦幫忙系列文、technice）。

### 3. 可选：尝试直连字幕（best-effort，失败就跳过，别卡住）
以下服务**在本环境大多被墙或已关停**，每个只给 ~20 秒，失败立即走第 2 步：
- `https://youtube-transcript.io/`、`https://youtubetranscript.com/`、`https://tactiq.io/tools/youtube-transcript`（字幕站，常被 YT 封或服务端转圈）
- 公共代理 `codetabs / corsfix / whateverorigin` 转发 `video.google.com/timedtext`（常被挡）
- 若已装 `agent-browser`，可试字幕站，但无头浏览器常被 Cloudflare 拦，别依赖
> 失败清单见 `references/workflow.md`，避免重复踩坑。

### 4. 交叉核对 + 综合
- 把多来源里的**章节时间戳、关键概念、命令/术语**对齐；冲突处以「官方简介 + 多笔记一致」为准。
- 若只有标题没有章节细节，就按视频主题用通用知识框架补全结构，并注明「章节细节依据有限，建议核对原片」。

### 5. 产出结构化总结
按 `references/summary_template.md` 的模板写 Markdown：基本信息 → 教学主线 → 分章节详细内容（带时间戳/要点/表格/命令）→ 核心速查表 → 小结 → **说明（来源与限制）**。

### 6. 落盘 + 呈现
- 写到当前工作目录，文件名如 `YouTube教学内容总结_<关键词>.md`。
- 用 present_files 呈现给用户。
- **必须诚实标注来源与限制**：若未能直取字幕，要写明「基于官方简介 + 多源笔记交叉核对，章节与要点已比对一致，但未逐字抓取字幕」。

### 7. 写运行日志（每次必做，不许省略）
在 `references/runlog.md` 的日志表末尾追加一行（格式见该文件）：视频ID、类型、noembed 是否命中、主路径是否命中、用了几篇来源、产出文件、用户反馈（未收到就写「未反馈」）、本次遇到的坑。**这些日志是本 skill 唯一的迭代原料。**

## 可靠性原则
- **不要因为抓不到字幕就报失败**：网页搜索 + 第三方笔记这条路，对热门教学视频基本够用。
- **语言跟随视频**：中文教学视频 → 中文总结；英文 → 英文（除非用户要求翻译）。
- **证据可搜到**：在「说明」里列出用到的来源链接，方便用户复查。
- 若用户能提供字幕文件（.srt/.vtt），优先按字幕做逐分钟精细拆解——这是最高保真路径。

## 与其它 skill 的配合
- 产出的总结文档，是「资产沉淀（zichan_chendian）」的天然内容资产。可在生成后建议用户用该 skill 把内容/素材/流量/信任四类沉淀下来，避免「写了就躺桌面」。

## 参考文件
- `references/workflow.md` —— 详细命令、可达/不可达端点清单、降级链
- `references/summary_template.md` —— 输出 Markdown 模板
- `references/runlog.md` —— 运行日志：运行前读（规避坑）、运行后写（迭代原料）
