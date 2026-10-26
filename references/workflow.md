# 工作流细节与降级链（实测于受限网络环境）

> 本文件记录「粘贴 YouTube 链接 → 生成教学内容总结」每一步的**实际可达端点**与**已证实失败的端点**，避免重复试错。

---

## A. 取元数据（Step 1）

**先规范化 URL（写死的规则，不许跳过）**：
- `https://www.youtube.com/shorts/<ID>` → `https://www.youtube.com/watch?v=<ID>`（noembed 不识别 shorts 形式，实测失效）
- 带多余参数的链接（`&list=`、`&t=` 等）先剥掉，只留 `watch?v=<ID>`
- 拿到 ID 后统一拼成 `https://www.youtube.com/watch?v=<ID>` 再调用下述端点

| 端点 | 命令 | 实测结果 |
|------|------|---------|
| noembed | `curl -sS -m 20 "https://noembed.com/embed?url=<URL>"` | ✅ 稳定返回 `{title, author_name, provider, thumbnail_url}`（watch 形式） |
| YouTube oembed | `curl -sS -m 20 "https://www.youtube.com/oembed?url=<URL>&format=json"` | ⚠️ 本环境常返回 401，作备选 |
| **兜底：WebSearch 搜视频 ID** | WebSearch `"视频ID"` 或完整 URL | ✅ 从搜索结果标题/摘要拿标题与频道（noembed/oembed 双失败时用，实测有效） |

拿到 `title` 后，它就是后续搜索的主关键词。

---

## B. 抓内容主路径（Step 2：WebSearch + WebFetch）

直连字幕在本环境基本不可行，这条是主力：

1. **WebSearch 查询组合**（语言跟随视频）：
   - `"<title>" 章节` / `"<title>" 教學 筆記` / `"<title>" 完整筆記`
   - `"<title>" summary` / `"<title>" notes` / `"<title>" chapters`
   - `"<title>" site:ithelp.ithome.com.tw` / `site:technice.com.tw`
   - 加视频 ID / 频道名一起搜提高命中
2. **WebFetch** 取 2–3 篇明确引用该视频的来源：
   - 官方视频简介里的章节表（最权威）
   - 同一作者配套文章
   - 第三方学习笔记（如 iT 邦幫忙 Day 系列、technice.com.tw）
3. 交叉核对章节时间戳、关键概念、命令。

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
  └─ 规范化：shorts/<ID> 与带参数链接 → watch?v=<ID>（写死，必做）
        └─ noembed 取 title/author
              ├─ 失败? 试 oembed
              │     └─ 仍失败? WebSearch 直接搜视频 ID/URL 拿标题（兜底，写死）
              └─ WebSearch 搜标题找章节+笔记
                    ├─ 找到 ≥2 篇引用源 → WebFetch 抓取 → 交叉核对 → 输出
                    └─ 找不到细节 → (best-effort) 试字幕站 20s → 仍无 → 用通用框架补全并标注限制
        └─ 用户给了 .srt/.vtt → 直接按字幕逐分钟拆解（最高保真）
```

## F. 运行日志（每次运行前后）

- **运行前**：读 `references/runlog.md` 的「已知经验」，规避已踩坑。
- **运行后**：向其日志表追加一行（格式见该文件）。连续同类问题 2–3 次 → 触发升版修复。
