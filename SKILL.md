---
name: youtube-teaching-summary
version: 1.2.0
display_name: YouTube 教学视频总结助手
display_name_en: YouTube Tutorial Summary Assistant
description: 输入一个 YouTube 或 B 站知识/教学/课程类视频链接，自动生成结构化的「学习视频教学内容总结」（分章节讲解 + 核心指令/概念速查表）。当直连视频平台或字幕代理受限时，自动改用「网页搜索 + 自动字幕站 + 多源交叉核对」的方式可靠产出。触发词：youtube 总结、b站总结、视频教学内容总结、教学视频笔记、根据视频生成总结、复制一个视频链接生成总结。
description_zh: 粘贴 YouTube 或 B 站教学/知识类视频链接，自动生成结构化学习笔记：分章节讲解、要点表格、核心指令/概念速查表；网络受限时自动降级为「网页搜索 + 自动字幕站 + 多源交叉核对」。
description_en: Paste a YouTube or Bilibili tutorial/educational video link and auto-generate a structured study note with chapter breakdowns, key-point tables, and a command/concept cheat sheet; falls back to web search, auto-caption sites, and multi-source cross-checking when direct access is restricted.
---

# YouTube 教学视频内容总结

把一个 YouTube 知识讲解视频，变成一份带章节、带要点、带速查表的结构化学习笔记。

## 何时使用
- 用户给了一个 YouTube 或 B 站视频链接，要求「总结 / 整理教学内容 / 做笔记 / 提取知识点」。
- 视频类型是：教程、技术讲解、公开课、软件教学、概念科普等「以传授知识为主」的内容。
- 用户想把看过的教学视频沉淀成可复习、可复用的文档。

## 核心工作流（按此顺序，别跳步）

### 0. 读运行日志（每次必做）
打开 `references/runlog.md` 通读「已知经验」与最近几条日志，规避已踩过的坑（如 Shorts 链接需先规范化）。**运行结束后回到第 7 步追加一条日志。**

### 1. 取元数据（按平台分支，几乎必定成功）
**先规范化 URL（写死，不许跳过）**：
- YouTube Shorts：`youtube.com/shorts/<ID>` → `youtube.com/watch?v=<ID>`（noembed 不识别 shorts 形式）
- YouTube 带参链接：剥掉 `&list=`、`&t=` 等多余参数，只留 `watch?v=<ID>`
- B 站：从 `bilibili.com/video/<BV号>` 提取 BV 号；`b23.tv` 短链先跟随跳转拿到 BV 号

**元数据获取（按平台）**：
- **YouTube**：
  ```bash
  curl -sS -m 20 "https://noembed.com/embed?url=<规范化后的URL>"      # 返回 title / author_name
  curl -sS -m 20 "https://www.youtube.com/oembed?url=<规范化后的URL>&format=json"  # 备选
  ```
  **限流防御（写死）**：批量调用时每请求间隔 3–5 秒；请求失败先 sleep 5s 重试**一次**，仍失败再走兜底——noembed 连发会被限流，重试即恢复（实测）。
  **兜底**：WebSearch 直接搜视频 ID/URL，从结果拿标题与频道（实测有效，如 noembed 403 的视频）。
- **B 站**（noembed 不支持，用官方 API，免鉴权）：
  ```bash
  curl -sS -m 20 "https://api.bilibili.com/x/web-interface/view?bvid=<BV号>" -H "User-Agent: Mozilla/5.0"
  ```
  返回 `data.title` / `data.owner.name` / `data.duration` 等（实测可达）。

拿到标题（title）、频道（author）。**标题是后面搜索的唯一关键词——严禁凭频道名推断视频主题**（实测教训：Mosh 的 React 教程被误当 Python 搜，结果全是频道其他视频）。

### 2. 抓内容（主路径 = 网页搜索，不要死磕直连字幕）
在受限网络里，直连 YouTube 与多数字幕代理经常失败。最稳的是**用精确标题去搜现成的章节表与学习笔记**。

**首选：自动字幕站（准字幕级来源，实测命中率极高）**——这类站自动从 captions 生成「章节+时间戳+摘要」，等于绕过字幕直连被墙的问题：
- `"<视频标题>" site:rosetta.to`（章节时间戳+金句）
- `glasp.co/youtube/<视频ID>`（**按 ID 建页**，直接拼 URL 试一次）
- `"<视频标题>" site:recapio.com`（带时间戳分段摘要）
- `"<视频标题>" site:sozai.app`（全文 transcript+章节）

**次选：人写笔记与官方资料**：
- `"<视频标题>" 章节` / `"<视频标题>" 教學 筆記` / `"<视频标题>" 完整筆記`
- `"<视频标题>" summary` / `"<视频标题>" notes` / `"<视频标题>" chapters`
- `"<视频标题>" site:ithelp.ithome.com.tw` / `site:technice.com.tw`（中文技术笔记站）
- 加上视频 ID 或频道名一起搜，提高命中

用 WebFetch 抓 2–3 篇来源，优先级：**官方视频简介章节表 > 自动字幕站 > 同一作者配套文章 > 第三方学习笔记**。

**同名甄别（写死）**：同一频道常有同名/近似名的多版本视频（如 Traversy《React JS Crash Course》有 2021/2024 两版，章节完全不同）。核对每个章节表来源的**视频 ID、时长、发布年份**，对不上就丢弃，宁缺毋滥。

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
