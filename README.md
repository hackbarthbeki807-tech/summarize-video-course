# summarize-video-course v2.8

一个用于把本地课程视频、字幕或逐字稿整理为可读知识资料的 Codex Skill。

v2.8 重点强化了课程笔记结构、案例复盘、金句原话边界、知识点来源标注、批次验收、设备适配和长任务监控。

## 公共版内容

```text
summarize-video-course/
├─ SKILL.md
├─ agents/openai.yaml
└─ references/approved-example-template.md
```

这个仓库只保存通用 Skill 规则，不保存任何真实课程内容。

## 明确不应上传的内容

不要把以下内容放入公开仓库：

- 课程视频、音频、字幕、逐字稿、逻辑稿、知识点卡片和课程成果；
- 真实课程名称、课时编号、课程文件夹、批次 manifest、运行日志和模型缓存；
- `media_index.json` 等包含真实文件名、时长、校验值或来源结构的文件；
- 真实课程范例、客户资料、绝对本地路径、API key、OAuth key、密码或其他凭据。

真实课程范例应保存在本地或私有项目中。通用 Skill 可以公开，课程资料不应随 Skill 一起公开。

## 安装

将本仓库克隆或下载到 Codex skills 目录：

```text
~/.codex/skills/summarize-video-course/
```

Windows 通常对应：

```text
C:\Users\<你的用户名>\.codex\skills\summarize-video-course\
```

安装后重新打开 Codex，或新建一个对话，然后使用：

```text
使用 $summarize-video-course 整理这批本地课程，并告诉我先看哪一篇。
```

## 本地处理边界

- 默认只读取本地课程源文件，不上传课程内容；
- 源视频保持只读，不复制、移动、重命名或删除；
- 临时媒体只能写入本地工作目录；
- 远程模型、付费 API 和云端存储不属于默认处理路径；
- 用户验收前不扩大批次，不把报告、文件夹或启动脚本当作课程文章。

## 发布前检查

公开仓库提交前，请确认仓库根目录和全部提交历史中都没有课程素材、真实范例、模型文件、本地路径或凭据。`.gitignore` 只能降低误提交概率，不能替代人工检查。

本仓库当前未指定开源许可证；如果希望网友合法修改、再发布或商用，请根据你的授权范围另行选择许可证。
