# YouTube 教学视频总结助手（youtube-teaching-summary）

> v1.0.1 ｜ 一个 WorkBuddy / Claude Code 等 Agent 通用的 Skill：**粘贴一个 YouTube 教学/知识类视频链接，自动产出结构化学习笔记**（分章节讲解 + 要点表格 + 核心指令/概念速查表）。

## 它解决什么问题

看完一个教程视频想记笔记，手动整理要花半小时。这个 Skill 把整条流程自动化：

```
YouTube 链接 ──► noembed 取标题/频道 ──► 网页搜索官方章节表 + 第三方笔记
             ──► 多源交叉核对 ──► 产出结构化 Markdown 总结（含速查表 + 来源说明）
```

特色：**网络受限环境可用**——直连 YouTube / 字幕代理失败时自动降级为「网页搜索 + 多源交叉核对」，并诚实标注来源与限制（参考 [references/workflow.md](references/workflow.md) 中的实测可达/不可达端点清单与降级决策树）。

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

支持普通视频与 Shorts。若你能提供字幕文件（.srt/.vtt），Skill 会优先按字幕做逐分钟精细拆解（最高保真路径）。

## 文件结构

```
├── SKILL.md                      # 触发条件 + 核心工作流 + 可靠性原则
└── references/
    ├── workflow.md               # 实测可达/不可达端点清单 + 降级决策树
    └── summary_template.md       # 输出 Markdown 模板
```

## 实测验证过的视频

- [PAPAYA 電腦教室：Git/GitHub 零基础入门](https://www.youtube.com/watch?v=FKXRiAiQFiY) → 产出 9 章节总结 + 15 条指令速查表
- [技术爬爬虾：Git+GitHub 核心概念大串讲（42min）](https://www.youtube.com/watch?v=bWUUHBVg-7E) → 产出 7 章节总结
- [Shorts：Hypit AI 克隆视频](https://www.youtube.com/shorts/3VD0-qerrOI) → 自动识别 Shorts、按流程拆解

## License

[MIT](LICENSE)
