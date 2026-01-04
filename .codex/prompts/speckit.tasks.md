---
description: 基于可用设计工件生成可执行且按依赖顺序排列的 tasks.md。
handoffs: 
  - label: 一致性分析
    agent: speckit.analyze
    prompt: 运行项目一致性分析
    send: true
  - label: 实施项目
    agent: speckit.implement
    prompt: 分阶段开始实现
    send: true
---

## 用户输入

```text
$ARGUMENTS
```

在继续之前你**必须**先考虑用户输入（如果不为空）。

## 大纲

1. **准备**：从仓库根目录运行 `.specify/scripts/bash/check-prerequisites.sh --json` 并解析 FEATURE_DIR 与 AVAILABLE_DOCS 列表。所有路径必须是绝对路径。对于参数中包含单引号（如 "I'm Groot"），使用转义语法：例如 'I'\''m Groot'（或尽量用双引号："I'm Groot"）。

2. **加载设计文档**：从 FEATURE_DIR 读取：
   - **必需**：plan.md（技术栈、库、结构），spec.md（带优先级的用户故事）
   - **可选**：data-model.md（实体）、contracts/（API 端点）、research.md（决策）、quickstart.md（测试场景）
   - 注意：并非所有项目都有全部文档。基于可用内容生成任务。

3. **执行任务生成流程**：
   - 加载 plan.md 并提取技术栈、库、项目结构
   - 加载 spec.md 并提取用户故事及其优先级（P1、P2、P3 等）
   - 若存在 data-model.md：提取实体并映射到用户故事
   - 若存在 contracts/：将端点映射到用户故事
   - 若存在 research.md：提取决策用于 setup 任务
   - 生成按用户故事组织的任务（参见下方规则）
   - 生成依赖图，展示用户故事完成顺序
   - 为每个用户故事生成并行执行示例
   - 验证任务完整性（每个用户故事拥有所需任务且可独立测试）

4. **生成 tasks.md**：使用 `.specify/templates/tasks-template.md` 作为结构并填充：
   - 从 plan.md 获取正确的功能名称
   - Phase 1：Setup 任务（项目初始化）
   - Phase 2：Foundational 任务（阻塞性前置条件）
   - Phase 3+：按 spec.md 的优先级为每个用户故事一阶段
   - 每阶段包含：故事目标、独立测试标准、测试（若请求）、实现任务
   - 最终阶段：Polish & Cross-Cutting Concerns
   - 所有任务必须符合严格清单格式（见下方规则）
   - 每个任务包含明确文件路径
   - Dependencies 章节展示故事完成顺序
   - 每个故事给出并行执行示例
   - Implementation strategy 章节（MVP 优先，渐进交付）

5. **汇报**：输出生成的 tasks.md 路径与摘要：
   - 任务总数
   - 每个用户故事的任务数
   - 并行机会
   - 每个故事的独立测试标准
   - 建议的 MVP 范围（通常只包含用户故事 1）
   - 格式校验：确认所有任务符合清单格式（复选框、ID、标签、路径）

任务生成上下文：$ARGUMENTS

生成的 tasks.md 必须可直接执行——每个任务要足够具体，使 LLM 无需额外上下文也能完成。

## 任务生成规则

**关键**：任务必须按用户故事组织，以便独立实现与测试。

**测试可选**：仅在功能规范明确要求或用户要求 TDD 时生成测试任务。

### 清单格式（必需）

每个任务必须严格遵循以下格式：

```text
- [ ] [TaskID] [P?] [Story?] 含文件路径的描述
```

**格式组成**：

1. **复选框**：始终以 `- [ ]` 开头
2. **任务 ID**：按执行顺序递增编号（T001、T002、T003...）
3. **[P] 标记**：仅当任务可并行（不同文件、无未完成依赖）时包含
4. **[Story] 标签**：仅对用户故事阶段任务必填
   - 格式：[US1]、[US2]、[US3] 等（映射到 spec.md 的用户故事）
   - Setup 阶段：不含故事标签
   - Foundational 阶段：不含故事标签
   - 用户故事阶段：必须有故事标签
   - Polish 阶段：不含故事标签
5. **描述**：清晰动作 + 具体文件路径

**示例**：

- ✅ 正确：`- [ ] T001 按实施计划创建项目结构`
- ✅ 正确：`- [ ] T005 [P] 在 src/middleware/auth.py 实现认证中间件`
- ✅ 正确：`- [ ] T012 [P] [US1] 在 src/models/user.py 创建 User 模型`
- ✅ 正确：`- [ ] T014 [US1] 在 src/services/user_service.py 实现 UserService`
- ❌ 错误：`- [ ] Create User model`（缺少 ID 和故事标签）
- ❌ 错误：`T001 [US1] Create model`（缺少复选框）
- ❌ 错误：`- [ ] [US1] Create User model`（缺少任务 ID）
- ❌ 错误：`- [ ] T001 [US1] Create model`（缺少文件路径）

### 任务组织

1. **来自用户故事（spec.md）** - 主要组织方式：
   - 每个用户故事（P1、P2、P3...）单独一阶段
   - 将相关组件映射到对应故事：
     - 该故事所需模型
     - 该故事所需服务
     - 该故事所需端点/UI
     - 若请求测试：该故事的测试任务
   - 标记故事依赖（大多数故事应独立）

2. **来自契约**：
   - 每个契约/端点 → 对应用户故事
   - 若请求测试：每个契约 → 在该故事阶段先做契约测试任务 [P]

3. **来自数据模型**：
   - 每个实体映射到需要它的用户故事
   - 若实体服务多个故事：放在最早的故事或 Setup 阶段
   - 关系 → 放在对应故事阶段的服务层任务

4. **来自 Setup/基础设施**：
   - 共享基础设施 → Setup 阶段（Phase 1）
   - 阻塞性任务 → Foundational 阶段（Phase 2）
   - 故事特定的 setup → 放在对应故事阶段

### 阶段结构

- **Phase 1**：Setup（项目初始化）
- **Phase 2**：Foundational（阻塞性前置条件 - 必须在用户故事前完成）
- **Phase 3+**：按优先级排列的用户故事（P1、P2、P3...）
  - 每个故事内：测试（若请求）→ 模型 → 服务 → 端点 → 集成
  - 每阶段应是可独立测试的增量
- **Final Phase**：Polish & Cross-Cutting Concerns
