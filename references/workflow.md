# 工作流细节与降级链（实测于受限网络环境）

> 本文件记录「粘贴 YouTube 链接 → 生成教学内容总结」每一步的**实际可达端点**与**已证实失败的端点**，避免重复试错。

---

## A. 取元数据（Step 1，按平台分支）

**先规范化 URL（写死的规则，不许跳过）**：
- `https://www.youtube.com/shorts/<ID>` → `https://www.youtube.com/watch?v=<ID>`（noembed 不识别 shorts 形式，实测失效）
- YouTube 带多余参数的链接（`&list=`、`&t=` 等）先剥掉，只留 `watch?v=<ID>`
- B 站：从 `bilibili.com/video/<BV号>` 提取 BV 号；`b23.tv` 短链先跟随跳转拿 BV 号
- YouTube 拿到 ID 后统一拼成 `https://www.youtube.com/watch?v=<ID>` 再调用下述端点

**YouTube 端点**：

| 端点 | 命令 | 实测结果 |
|------|------|---------|
| noembed | `curl -sS -m 20 "https://noembed.com/embed?url=<URL>"` | ✅ 稳定返回 `{title, author_name}`（watch 形式） |
| YouTube oembed | `curl -sS -m 20 "https://www.youtube.com/oembed?url=<URL>&format=json"` | ⚠️ 本环境常返回 401，作备选 |
| **限流防御** | 批量调用每请求间隔 3–5s；失败先 sleep 5s 重试一次 | ✅ 实测：连发第 6 个请求起超时，重试即恢复 |
| **兜底：WebSearch 搜视频 ID** | WebSearch `"视频ID"` 或完整 URL | ✅ noembed 403（如 T93SayXhw2w）时有效 |

**B 站端点（v1.2.0 新增）**：

| 端点 | 命令 | 实测结果 |
|------|------|---------|
| noembed | 同上格式 | ❌ `no matching providers found`，不支持 B 站 |
| **B 站官方 API（唯一路径）** | `curl -sS -m 20 "https://api.bilibili.com/x/web-interface/view?bvid=<BV号>" -H "User-Agent: Mozilla/5.0"` | ✅ **免鉴权可达**，返回 `data.title` / `data.owner.name` / `data.duration` |
| **多 P 目录（v1.3.0）** | 同上 view API 的 `data.pages` 数组 | ✅ 实测 BV12J41137hu 返回 **80 个分 P**，每项含 `page`/`part`（标题）/`cid`/`duration`——系列总结的现成目录 |
| **字幕级探测（v1.3.0，best-effort）** | `curl -sS -m 20 "https://api.bilibili.com/x/player/wbi/v2?bvid=<BV号>&cid=<cid>" -H "User-Agent: Mozilla/5.0" -H "Referer: https://www.bilibili.com"` | 🔍 有线索待验证（BiliTools/Bili Note 开源工具实证 B 站有公开字幕与 AI 摘要接口）；`data.subtitle.subtitles` 非空则取 `subtitle_url` 下载 JSON 字幕，空即跳过 |

拿到 `title` 后，它就是后续搜索的主关键词。

---

## B. 抓内容主路径（Step 2：WebSearch + WebFetch）

直连字幕在本环境基本不可行，这条是主力。

**搜索关键词铁律**：必须用**精确标题**（来自 Step 1），严禁凭频道名推断视频主题（实测教训：Mosh 的 React 教程被误当 Python 搜，结果全是频道其他视频的章节表）。

1. **首选查询组合——自动字幕站（准字幕级，v1.2.0 升级为主路径首选）**：
   - `"<title>" site:rosetta.to`（章节时间戳 + 金句 + Q&A）
   - `glasp.co/youtube/<视频ID>`（按 ID 建页，可直接拼 URL 试）
   - `"<title>" site:recapio.com`（带时间戳分段摘要）
   - `"<title>" site:sozai.app`（全文 transcript + 章节）
   - 这类站自动从 captions 生成内容，热门英文视频命中率极高，等于绕过字幕直连被墙
2. **次选查询组合——人写笔记与官方资料**（语言跟随视频）：
   - `"<title>" 章节` / `"<title>" 教學 筆記` / `"<title>" 完整筆記`
   - `"<title>" summary` / `"<title>" notes` / `"<title>" chapters`
   - `"<title>" site:ithelp.ithome.com.tw` / `site:technice.com.tw`
   - 加视频 ID / 频道名一起搜提高命中
3. **WebFetch** 取 2–3 篇来源，优先级：**官方简介章节表 > 自动字幕站 > 同作者配套文章 > 第三方笔记**
4. **同名甄别（写死）**：同一频道常有同名多版本（Traversy《React JS Crash Course》2021/2024 两版章节完全不同）——核对每个来源的**视频 ID、时长、发布年份**，对不上就丢弃
5. 交叉核对章节时间戳、关键概念、命令。

---

## C. 直连字幕（Step 3：best-effort，失败即跳过）

| 端点 / 服务 | 实测结果 | 备注 |
|------|---------|------|
| `https://www.youtube.com/watch?v=...` 直连 | ❌ 401 / 超时 | 本环境无直连 |
| `https://www.youtube.com/oembed` | ❌ 401 | 见 A |
| `https://cobalt.tools/api/json` | ❌ 已关停 | 返回空 |
| `https://youtube-transcript.io/` | ⚠️ 服务端转圈 | 其服务器也被 YT 挡，无结果 |
| `https://youtubetranscript.com/?server_vid2=ID` | ❌ YT 拦截 | caption 代理被封 |
| `https://video.google.com/timedtext` | ❌ 被挡 | 经典端点失效 |
| 公共代理 codetabs / corsfix / whateverorigin | ❌ 被挡 | 转发 YT 全失败 |
| Invidious / Piped 公共实例 | ❌ 不可达 | 全部超时 |
| `agent-browser` 无头浏览器跑字幕站 | ⚠️ Cloudflare 拦 | 即使装了浏览器也常被 Turnstile 挡，别依赖 |

**结论**：直连字幕链路在本环境不可靠，**不要把它当主路径**，超时立即回退到 B（WebSearch）。

---

## D. 综合与输出（Step 4–6）

- 多源对齐章节、概念、命令；冲突以「官方简介 + 多笔记一致」为准。
- 只有标题无细节时，用通用知识框架补全结构，并在说明里标注「建议核对原片」。
- 输出模板见 `summary_template.md`；落盘后 present_files 呈现，并诚实标注来源与限制。

---

## E. 降级决策树（速查）

```
拿到 URL
  └─ 识别平台：YouTube（含 shorts）或 B 站（BV号 / b23.tv）
        ├─ YouTube ── 规范化 → watch?v=<ID>（写死，必做）
        │     └─ noembed（批量间隔 3–5s；失败 sleep 5s 重试一次）
        │           ├─ 失败? 试 oembed
        │           │     └─ 仍失败? WebSearch 直接搜视频 ID/URL 拿标题（兜底，写死）
        │           └─ WebSearch：先自动字幕站 → 再人写笔记
        │                 ├─ 找到 ≥2 篇来源（核对 ID/时长/年份）→ WebFetch → 交叉核对 → 输出
        │                 └─ 找不到细节 → (best-effort) 试字幕站 20s → 仍无 → 通用框架补全并标注限制
        ├─ B 站 ── 提取 BV号
        │     └─ 官方 API 取 title/owner/pages（免鉴权，实测可达）
        │           ├─ pages >1? → 系列视频模式（见 G 节）：总表 + 与用户对齐范围 + 逐讲
        │           ├─ (best-effort) player/wbi/v2 字幕探测 → 有字幕则字幕级输出
        │           └─ WebSearch 搜标题找章节+笔记 → 同上交叉核对 → 输出
        └─ YouTube 播放列表（playlist?list=...）
              └─ noembed 实证不支持 → WebSearch 找第三方整理的列表
                    ├─ 找到 → 按列表逐集走单视频流程
                    └─ 找不到 → 说明限制 + 列出已知前几集 + 问用户
  └─ 用户给了 .srt/.vtt → 直接按字幕逐分钟拆解（最高保真）
```

## F. 运行日志（每次运行前后）

- **运行前**：读 `references/runlog.md` 的「已知经验」，规避已踩坑。
- **运行后**：向其日志表追加一行（格式见该文件）。连续同类问题 2–3 次 → 触发升版修复。

---

## G. 系列视频模式（v1.3.0）

**B 站多 P（首选路径，数据现成）**：
1. view API 的 `pages` 数组即全量目录：`page`（P号）/`part`（标题）/`cid`/`duration`（实测 80 P 全返回）。
2. 分 P 多于 10 个 → 先产出「学习路径总表」再与用户对齐范围，勿擅自全量跑。
3. 逐讲搜索词：`"<合集标题> P<n> <分P标题>"`（B 站教学合集在 CSDN/博客园常有逐讲笔记）。
4. 产出：每讲一份 `<合集名>_P<n>_<分P标题>.md` + 一份总表 `<合集名>_学习路径总表.md`（P号｜标题｜时长｜核心主题｜依赖前置｜一句话收获 + 建议学习顺序短评）。

**YouTube 播放列表（实证受限）**：
- noembed 对 `playlist?list=<PLID>` 返回 `no matching providers`——无直接列表来源。
- 策略：WebSearch 搜播放列表标题/ID + `playlist 章节 OR 所有视频` 找第三方整理；带 `&list=` 参数的**单视频**链接不受影响（规范化规则已覆盖，剥掉 list 参数即可）。

**runlog**：系列模式每讲各记一行，另加一行 `series` 类型记录整体。
