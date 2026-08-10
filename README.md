# summarize-video-course

一个用于把本地课程视频、字幕或逐字稿整理为可读知识资料的 Codex Skill。

## 内容

```text
summarize-video-course/
├─ SKILL.md
├─ agents/openai.yaml
└─ references/approved-example-template.md
```

这个导出包不包含真实课程视频、音频、字幕、逐字稿、模型文件或课程成果。原项目中的真实课程范例没有放入这里，以免把课程内容公开。

## 安装

把 `summarize-video-course` 文件夹复制到 Codex skills 目录，例如：

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

## GitHub 上传

推荐把本目录作为一个独立仓库的根目录上传：

```powershell
cd github-export\summarize-video-course
git init
git add .
git commit -m "feat: add summarize-video-course skill"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

如果仓库公开，上传前请再次确认没有课程内容、客户资料、模型文件、API key、OAuth key 或本地绝对路径。

## 私有参考范例

如果需要保留真实课程范例，请放在私有仓库或本地 skill 目录的 `references/` 中，不要放进公开仓库。`SKILL.md` 只要求模仿结构和表达密度，不能把范例事实当成新课程内容。
