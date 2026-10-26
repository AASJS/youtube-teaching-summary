# YouTube 教学视频总结助手（youtube-teaching-summary）

> v1.2.0 ｜ 一个 WorkBuddy / Claude Code 等 Agent 通用的 Skill：**粘贴一个 YouTube / B 站教学/知识类视频链接，自动产出结构化学习笔记**（分章节讲解 + 要点表格 + 核心指令/概念速查表）。

## 它解决什么问题

看完一个 40 分钟的教程视频想记笔记：暂停、回退、截图、敲字……手动整理通常要花 **30 分钟起步**，而且经常「记了一半就弃疗」。这个 Skill 把整条流程压缩成「粘贴链接 → 等 2 分钟 → 拿到笔记」：

```
YouTube 链接 ──► noembed 取标题/频道（秒回）
             ──► 网页搜索官方章节表 + 第三方学习笔记
             ──► 多源交叉核对（章节/命令/概念对齐）
             ──► 产出结构化 Markdown 总结：分章节讲解 + 要点表格 + 指令速查表 + 来源说明
```

两个特色：

1. **网络受限环境可用**。直连 YouTube、字幕代理站在很多网络下全部失败——本 Skill 不死磕直连，主路径就是「网页搜索 + 多源交叉核对」，对热门教学视频命中率极高。完整降级决策树见 [references/workflow.md](references/workflow.md)（含实测可达/不可达端点清单，避免重复踩坑）。
2. **诚实标注来源与限制**。产出末尾固定带「说明」段：列出来源链接；若未能抓到字幕原文，会明确写「基于官方简介 + 多源笔记交叉核对，未逐字核对字幕」，绝不假装看过原片。

## 安装

**方式一：手动安装（通用）**

```bash
# 克隆本仓库
git clone https://github.com/AASJS/youtube-teaching-summary.git

# 复制到用户级 skills 目录（所有项目可用）
# Windows
xcopy /E /I youtube-teaching-summary "%USERPROFILE%\.workbuddy\skills\youtube-teaching-summary"
# macOS / Linux
cp -r youtube-teaching-summary ~/.workbuddy/skills/

# 重启 / 刷新 WorkBuddy 即可
```

**方式二：skills CLI（如已安装）**

```bash
npx skills add AASJS/youtube-teaching-summary -g
```

## 使用

对 Agent 说任意一句：

- 「总结这个视频：https://www.youtube.com/watch?v=xxxx」
- 「https://www.youtube.com/shorts/xxxx 帮我整理成笔记」
- 「把这条视频的教学内容总结出来」

支持 YouTube（普通视频 / Shorts）与 **B 站（BV号 / b23.tv 短链）**。若你能提供字幕文件（.srt/.vtt），Skill 会优先按字幕做逐分钟精细拆解（最高保真路径）。

```
「总结这个视频：https://www.youtube.com/watch?v=xxxx」
「https://www.youtube.com/shorts/xxxx 帮我整理成笔记」
「https://www.bilibili.com/video/BVxxxx 这个视频的笔记帮我做好吗」
```

## 文件结构

```
├── SKILL.md                      # 触发条件 + 核心工作流 + 可靠性原则
└── references/
    ├── workflow.md               # 实测可达/不可达端点清单 + 降级决策树
    ├── summary_template.md       # 输出 Markdown 模板
    └── runlog.md                 # 运行日志：运行前读（规避坑）、运行后写（迭代原料）
```

## 实测验证过的视频

- [PAPAYA 電腦教室：Git/GitHub 零基础入门](https://www.youtube.com/watch?v=FKXRiAiQFiY) → 产出 9 章节总结 + 15 条指令速查表
- [技术爬爬虾：Git+GitHub 核心概念大串讲（42min）](https://www.youtube.com/watch?v=bWUUHBVg-7E) → 产出 7 章节总结
- [Shorts：Hypit AI 克隆视频](https://www.youtube.com/shorts/3VD0-qerrOI) → 自动识别 Shorts、按流程拆解

## License

[MIT](LICENSE)
