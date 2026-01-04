---
description: 使用计划模板执行实施规划流程并生成设计工件。
handoffs: 
  - label: 创建任务
    agent: speckit.tasks
    prompt: 将计划拆分为任务
    send: true
  - label: 创建清单
    agent: speckit.checklist
    prompt: 为以下领域创建清单...
---

## 用户输入

```text
$ARGUMENTS
```

在继续之前你**必须**先考虑用户输入（如果不为空）。

## 大纲

1. **准备**：从仓库根目录运行 `.specify/scripts/bash/setup-plan.sh --json` 并解析 JSON 中的 FEATURE_SPEC、IMPL_PLAN、SPECS_DIR、BRANCH。对于参数中包含单引号（如 "I'm Groot"），使用转义语法：例如 'I'\''m Groot'（或尽量用双引号："I'm Groot"）。

2. **加载上下文**：读取 FEATURE_SPEC 与 `.specify/memory/constitution.md`。加载 IMPL_PLAN 模板（已复制）。

3. **执行计划流程**：按 IMPL_PLAN 模板结构完成：
   - 填写 Technical Context（未知项标记为 "NEEDS CLARIFICATION"）
   - 用宪法填充 Constitution Check 小节
   - 评估门禁（如违反且无正当理由则 ERROR）
   - Phase 0：生成 research.md（解决所有 NEEDS CLARIFICATION）
   - Phase 1：生成 data-model.md、contracts/、quickstart.md
   - Phase 1：运行代理脚本更新上下文
   - 设计完成后重新评估 Constitution Check

4. **停止并汇报**：命令在 Phase 2 规划结束后停止。汇报分支、IMPL_PLAN 路径与生成的工件。

## 阶段

### Phase 0：大纲与调研

1. **从 Technical Context 抽取未知项**：
   - 每个 NEEDS CLARIFICATION → 调研任务
   - 每个依赖 → 最佳实践任务
   - 每个集成 → 模式调研任务

2. **生成并派发调研代理**：

   ```text
   对于 Technical Context 中每个未知项：
     任务：“为 {feature context} 调研 {unknown}”
   对于每个技术选择：
     任务：“在 {domain} 中查找 {tech} 的最佳实践”
   ```

3. **在 `research.md` 中汇总发现**，格式：
   - 决策：[选择了什么]
   - 理由：[为何选择]
   - 备选方案：[评估过的替代方案]

**输出**：research.md，所有 NEEDS CLARIFICATION 已解决

### Phase 1：设计与契约

**前置条件**：`research.md` 完成

1. **从功能规范提取实体** → `data-model.md`：
   - 实体名称、字段、关系
   - 来自需求的校验规则
   - 适用时包含状态迁移

2. **从功能需求生成 API 契约**：
   - 每个用户动作 → 一个端点
   - 使用标准 REST/GraphQL 模式
   - 输出 OpenAPI/GraphQL schema 到 `/contracts/`

3. **代理上下文更新**：
   - 运行 `.specify/scripts/bash/update-agent-context.sh codex`
   - 这些脚本会检测当前使用的 AI 代理
   - 更新对应的代理上下文文件
   - 只添加当前计划中的新技术
   - 保留标记之间的手工内容

**输出**：data-model.md、/contracts/*、quickstart.md、代理特定文件

## 关键规则

- 使用绝对路径
- 门禁失败或存在未解决的澄清则 ERROR
