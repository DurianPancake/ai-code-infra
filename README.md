# ai-code-infra

AI 编程工作流的基础设施仓库，存放自用的 Claude Code Skills、提示词组件和知识文档。

## 目录结构

```
ai-code-infra/
└── skills/          # Claude Code 自定义 Skills
    └── github-pr/   # GitHub PR 创建工作流
```

## Skills

Skills 是 Claude Code 的自定义工作流指令，通过结构化提示词驱动 AI 完成特定任务。

| Skill | 描述 |
|-------|------|
| [github-pr](skills/github-pr/SKILL.md) | 向 GitHub 仓库创建 Pull Request，支持 fork → upstream 跨仓库 PR，完成从材料收集到 PR 链接生成的完整流程 |

## 使用方法

### 安装 Skill

将 `skills/<skill-name>/SKILL.md` 的内容注册到 Claude Code 的 skills 配置中。

具体配置方式参考 [Claude Code 官方文档](https://docs.anthropic.com/claude/docs)。

### 调用 Skill

在 Claude Code 对话中使用自然语言触发，例如：

```
创建PR
提交到上游
pull request
```

## 友链

- [LINUX DO](https://linux.do/)

## 贡献

欢迎提交 Issue 和 PR。
