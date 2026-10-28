---
type: workflow
created: "2026-10-28"
updated: "2026-10-28"
tags: [knowledge-management, workflow]
---

# 资料整理 SOP

本流程适用于 Literature 知识库：搜集外部资料 → 发送材料或主题 → 选择模板 → 整理知识 → 写入对应目录。

## 1. 搜集与发送

可以自行搜集后发送，也可以直接委托 AI 搜集。

| 入口 | 提供什么 | 处理方式 |
|---|---|---|
| 已有资料 | 粘贴正文、附件或链接 | 读取材料并提炼知识；需要时补充核查 |
| 只有问题或主题 | 问题、关注范围；可补充优先来源 | 先检索并阅读相关资料，再整理笔记 |

自行收集时，尽量同时保留标题、作者和原始链接；论文可附 DOI，版本敏感的文档注明日期或版本。网页受登录限制时，直接附正文或文件即可，不必为了整理笔记在项目中另存 TXT。

把材料发送到以 Literature 为工作目录的聊天中。可选信息包括：阅读目的、偏好模板、篇幅、重点和是否允许补充检索。无需每次把所有字段填满；已经明确的信息不必重复。

## 2. 交给 AI 执行

常用指令：

```text
按 Literature 的资料整理流程，将附件整理为笔记，选择合适模板并写入对应目录。
```

指定模板：

```text
使用 $literature-notes，把这些材料整理为 comparison 笔记，重点比较适用条件与局限，并归档。
```

委托搜集：

```text
使用 $literature-notes，搜集关于【主题】的一手资料，回答【具体问题】，整理成笔记并归档。
```

限定材料范围：

```text
使用 $literature-notes，仅依据所给材料整理，不联网补充；无法确认的关键内容标明存疑。
```

AI 根据笔记目的选择项目模板、确定文件名与目录并直接写入。多篇材料不自动等于 comparison；没有特定偏好时，让 AI 自行选择即可。模板与目录的对应规则见 [skill 的“选择模板和目标目录”](../.agents/skills/literature-notes/SKILL.md#2-选择模板和目标目录)。

## 3. 查看与修订

收到文件链接后，在笔记中检查核心问题是否讲清、对比或操作是否可用、关键结论是否保留条件。需要调整时直接描述目标，例如“缩短背景”“补充适用边界”“删除弱关联”，AI 在原笔记中修订。

交付以归档后的 Markdown 文件为准。具体写作、来源处理、知识网络与交付检查规则统一维护在 [Literature 资料笔记 skill](../.agents/skills/literature-notes/SKILL.md)，避免 SOP 和执行规则分别修改后不一致。

## 4. 本项目如何使用这套规则

- [AGENTS.md](../AGENTS.md) 是项目入口，在相关任务中引导读取 skill。
- [.agents/skills/literature-notes/SKILL.md](../.agents/skills/literature-notes/SKILL.md) 是 AI 的执行规则，内容与产物范围均限于本知识库。
- [99-Templates](../99-Templates/) 是模板来源；按任务读取，保留适用内容并调整结构。

skill 按 Codex 的项目级目录组织，未安装到用户全局目录。若当前聊天尚未列出新 skill，可直接要求“读取项目的 `.agents/skills/literature-notes/SKILL.md` 后执行”；若选择器未更新，可重启 Codex 再查看。[官方说明](https://learn.chatgpt.com/docs/build-skills)
