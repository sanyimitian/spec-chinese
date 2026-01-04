---
description: 基于现有设计产物，将 tasks 转换为可执行、按依赖排序的 GitHub issues。
tools: ['github/github-mcp-server/issue_write']
---

## 用户输入（User Input）

```text
$ARGUMENTS
```

在继续之前，你**必须**先考虑用户输入（若不为空）。

## 流程概览（Outline）

1. 在仓库根目录运行 `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks`，解析 FEATURE_DIR 与 AVAILABLE_DOCS。所有路径必须为绝对路径。参数含单引号（如 "I'm Groot"）时使用转义：例如 `I'\''m Groot`（或尽量用双引号："I'm Groot"）。
1. 从脚本输出中提取 **tasks** 的路径。
1. 运行以下命令获取 Git remote：

```bash
git config --get remote.origin.url
```

> [!CAUTION]
> 仅当 remote 是 GitHub URL 时才继续下一步

1. 对任务列表中的每个任务，使用 GitHub MCP server 在与该 remote 对应的仓库中创建 issue。

> [!CAUTION]
> 任何情况下都不得在与 remote URL 不匹配的仓库里创建 issue
