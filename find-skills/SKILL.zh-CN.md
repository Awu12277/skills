---
name: find-skills
description: 当用户询问"如何做X"、"找一下X的技能"、"有没有能做...的技能"，或表示有兴趣扩展智能体能力时，帮助用户发现和安装技能。当用户寻找可能已作为可安装技能存在的功能时，应使用此技能。
---

# 查找技能

此技能帮助你从开放的智能体技能生态系统中发现和安装技能。

## 何时使用此技能

当用户符合以下情况时使用此技能：

- 询问"如何做X"，其中X可能是已有技能支持的常见任务
- 说"找一下X的技能"或"有没有X的技能"
- 询问"你能做X吗"，其中X是一种专业能力
- 表示有兴趣扩展智能体能力
- 想要搜索工具、模板或工作流程
- 提到希望在特定领域获得帮助（设计、测试、部署等）

## Skills CLI 是什么？

Skills CLI（`npx skills`）是开放智能体技能生态系统的包管理器。技能是以专业知识、工作流程和工具来扩展智能体能力的模块化包。

**主要命令：**

- `npx skills find [query]` - 交互式或按关键词搜索技能
- `npx skills add <package>` - 从 GitHub 或其他来源安装技能
- `npx skills check` - 检查技能更新
- `npx skills update` - 更新所有已安装的技能

**浏览技能请访问：** https://skills.sh/

## 如何帮助用户查找技能

### 第一步：了解用户需求

当用户请求帮助时，明确以下内容：

1. 领域（例如 React、测试、设计、部署）
2. 具体任务（例如编写测试、创建动画、审查 PR）
3. 这是否是足够常见的任务，以至于可能存在相应技能

### 第二步：先查看排行榜

在运行 CLI 搜索之前，先查看 [skills.sh 排行榜](https://skills.sh/)，确认该领域是否已有知名技能。排行榜按总安装量对技能进行排名，展示最受欢迎和经过实战检验的选项。

例如，Web 开发领域的热门技能包括：
- `vercel-labs/agent-skills` — React、Next.js、Web 设计（每个 100K+ 安装量）
- `anthropics/skills` — 前端设计、文档处理（100K+ 安装量）

### 第三步：搜索技能

如果排行榜没有覆盖用户需求，运行查找命令：

```bash
npx skills find [query]
```

例如：

- 用户问"如何让我的 React 应用更快？" → `npx skills find react performance`
- 用户问"你能帮我审查 PR 吗？" → `npx skills find pr review`
- 用户说"我需要创建一个变更日志" → `npx skills find changelog`

### 第四步：推荐前先验证质量

**不要仅根据搜索结果就推荐技能。** 务必验证以下几点：

1. **安装量** — 优先选择安装量 1K+ 的技能。对 100 以下的技能要谨慎。
2. **来源可靠性** — 官方来源（`vercel-labs`、`anthropics`、`microsoft`）比未知作者更值得信赖。
3. **GitHub 星标** — 检查源代码仓库。来自星标少于 100 的仓库的技能应持谨慎态度。

### 第五步：向用户展示选项

找到相关技能后，向用户提供以下信息：

1. 技能名称及其功能
2. 安装量和来源
3. 用户可运行的安装命令
4. skills.sh 上的详细链接

回复示例：

```
我找到了一个可能有帮助的技能！"react-best-practices" 技能提供来自 Vercel 工程团队的 React 和 Next.js 性能优化指南。
（185K 安装量）

安装方式：
npx skills add vercel-labs/agent-skills@react-best-practices

了解更多：https://skills.sh/vercel-labs/agent-skills/react-best-practices
```

### 第六步：提供安装帮助

如果用户想继续，你可以为他们安装技能：

```bash
npx skills add <owner/repo@skill> -g -y
```

`-g` 标志表示全局安装（用户级别），`-y` 跳过确认提示。

## 常见技能分类

搜索时可考虑以下常见分类：

| 分类           | 示例查询                                    |
| -------------- | ------------------------------------------- |
| Web 开发       | react, nextjs, typescript, css, tailwind    |
| 测试           | testing, jest, playwright, e2e              |
| DevOps         | deploy, docker, kubernetes, ci-cd           |
| 文档           | docs, readme, changelog, api-docs           |
| 代码质量       | review, lint, refactor, best-practices      |
| 设计           | ui, ux, design-system, accessibility        |
| 生产力         | workflow, automation, git                   |

## 有效搜索技巧

1. **使用具体关键词**："react testing" 比单独搜索 "testing" 效果更好
2. **尝试替代术语**：如果 "deploy" 不起作用，试试 "deployment" 或 "ci-cd"
3. **检查热门来源**：许多技能来自 `vercel-labs/agent-skills` 或 `ComposioHQ/awesome-claude-skills`

## 未找到技能时

如果没有相关技能：

1. 承认未找到现有技能
2. 提出直接使用通用能力帮助用户完成任务
3. 建议用户可以使用 `npx skills init` 创建自己的技能

示例：

```
我搜索了与"xyz"相关的技能，但没有找到匹配项。
我仍然可以直接帮助你完成这项任务！你想让我继续吗？

如果这是你经常做的事情，可以创建自己的技能：
npx skills init my-xyz-skill
```
