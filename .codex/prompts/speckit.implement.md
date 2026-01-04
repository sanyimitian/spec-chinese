---
description: 处理并执行 tasks.md 中定义的全部任务，完成实施计划。
---

## 用户输入

```text
$ARGUMENTS
```

在继续之前你**必须**先考虑用户输入（如果不为空）。

## 大纲

1. 从仓库根目录运行 `.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` 并解析 FEATURE_DIR 与 AVAILABLE_DOCS 列表。所有路径必须是绝对路径。对于参数中包含单引号（如 "I'm Groot"），使用转义语法：例如 'I'\''m Groot'（或尽量用双引号："I'm Groot"）。

2. **检查清单状态**（若存在 FEATURE_DIR/checklists/）：
   - 扫描 checklists/ 目录下的所有清单文件
   - 对每个清单统计：
     - 总项：匹配 `- [ ]` 或 `- [X]` 或 `- [x]` 的所有行
     - 已完成：匹配 `- [X]` 或 `- [x]` 的行
     - 未完成：匹配 `- [ ]` 的行
   - 生成状态表：

     ```text
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```

   - 计算总体状态：
     - **PASS**：所有清单未完成项为 0
     - **FAIL**：至少一个清单存在未完成项

   - **若存在未完成清单**：
     - 显示表格与未完成数量
     - **停止**并询问：“有些清单未完成。仍要继续实施吗？(yes/no)”
     - 等待用户回复后再继续
     - 若用户说 "no" / "wait" / "stop"，停止执行
     - 若用户说 "yes" / "proceed" / "continue"，继续步骤 3

   - **若所有清单完成**：
     - 显示所有清单通过的表格
     - 自动进入步骤 3

3. 加载并分析实施上下文：
   - **必须**：读取 tasks.md 获取完整任务清单与执行计划
   - **必须**：读取 plan.md 获取技术栈、架构与文件结构
   - **如存在**：读取 data-model.md 获取实体与关系
   - **如存在**：读取 contracts/ 获取 API 规范与测试要求
   - **如存在**：读取 research.md 获取技术决策与约束
   - **如存在**：读取 quickstart.md 获取集成场景

4. **项目设置校验**：
   - **必须**：基于实际项目设置创建/验证忽略文件：

   **检测与创建逻辑**：
   - 若以下命令成功，视为 git 仓库（创建/验证 .gitignore）：

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```

   - 若存在 Dockerfile* 或 plan.md 中涉及 Docker → 创建/验证 .dockerignore
   - 若存在 .eslintrc* → 创建/验证 .eslintignore
   - 若存在 eslint.config.* → 确保配置的 `ignores` 覆盖必要模式
   - 若存在 .prettierrc* → 创建/验证 .prettierignore
   - 若存在 .npmrc 或 package.json → 创建/验证 .npmignore（若发布包）
   - 若存在 terraform 文件（*.tf）→ 创建/验证 .terraformignore
   - 若需要 .helmignore（存在 helm chart）→ 创建/验证 .helmignore

   **若忽略文件已存在**：验证是否包含关键模式，仅追加缺失的关键模式
   **若忽略文件缺失**：按检测到的技术创建完整模式集合

   **按技术栈的常见模式**（来自 plan.md）：
   - **Node.js/JavaScript/TypeScript**：`node_modules/`、`dist/`、`build/`、`*.log`、`.env*`
   - **Python**：`__pycache__/`、`*.pyc`、`.venv/`、`venv/`、`dist/`、`*.egg-info/`
   - **Java**：`target/`、`*.class`、`*.jar`、`.gradle/`、`build/`
   - **C#/.NET**：`bin/`、`obj/`、`*.user`、`*.suo`、`packages/`
   - **Go**：`*.exe`、`*.test`、`vendor/`、`*.out`
   - **Ruby**：`.bundle/`、`log/`、`tmp/`、`*.gem`、`vendor/bundle/`
   - **PHP**：`vendor/`、`*.log`、`*.cache`、`*.env`
   - **Rust**：`target/`、`debug/`、`release/`、`*.rs.bk`、`*.rlib`、`*.prof*`、`.idea/`、`*.log`、`.env*`
   - **Kotlin**：`build/`、`out/`、`.gradle/`、`.idea/`、`*.class`、`*.jar`、`*.iml`、`*.log`、`.env*`
   - **C++**：`build/`、`bin/`、`obj/`、`out/`、`*.o`、`*.so`、`*.a`、`*.exe`、`*.dll`、`.idea/`、`*.log`、`.env*`
   - **C**：`build/`、`bin/`、`obj/`、`out/`、`*.o`、`*.a`、`*.so`、`*.exe`、`Makefile`、`config.log`、`.idea/`、`*.log`、`.env*`
   - **Swift**：`.build/`、`DerivedData/`、`*.swiftpm/`、`Packages/`
   - **R**：`.Rproj.user/`、`.Rhistory`、`.RData`、`.Ruserdata`、`*.Rproj`、`packrat/`、`renv/`
   - **通用**：`.DS_Store`、`Thumbs.db`、`*.tmp`、`*.swp`、`.vscode/`、`.idea/`

   **工具特定模式**：
   - **Docker**：`node_modules/`、`.git/`、`Dockerfile*`、`.dockerignore`、`*.log*`、`.env*`、`coverage/`
   - **ESLint**：`node_modules/`、`dist/`、`build/`、`coverage/`、`*.min.js`
   - **Prettier**：`node_modules/`、`dist/`、`build/`、`coverage/`、`package-lock.json`、`yarn.lock`、`pnpm-lock.yaml`
   - **Terraform**：`.terraform/`、`*.tfstate*`、`*.tfvars`、`.terraform.lock.hcl`
   - **Kubernetes/k8s**：`*.secret.yaml`、`secrets/`、`.kube/`、`kubeconfig*`、`*.key`、`*.crt`

5. 解析 tasks.md 结构并提取：
   - **任务阶段**：Setup、Tests、Core、Integration、Polish
   - **任务依赖**：顺序执行 vs 并行规则
   - **任务细节**：ID、描述、文件路径、并行标记 [P]
   - **执行流**：顺序与依赖要求

6. 按任务计划执行实施：
   - **分阶段执行**：每阶段完成后再进入下一阶段
   - **遵守依赖**：顺序任务按顺序执行，并行任务 [P] 可并行
   - **遵循 TDD**：测试任务先于对应实现任务
   - **基于文件的协调**：涉及同一文件的任务必须顺序执行
   - **验证检查点**：每阶段完成后进行验证

7. 实施执行规则：
   - **先做 Setup**：初始化项目结构、依赖与配置
   - **测试先行**：当需要为契约、实体与集成场景写测试时
   - **核心开发**：实现模型、服务、CLI 命令、端点
   - **集成工作**：数据库连接、中间件、日志、外部服务
   - **打磨与验证**：单元测试、性能优化、文档

8. 进度跟踪与错误处理：
   - 每完成一个任务报告进度
   - 若任何非并行任务失败，停止执行
   - 对于并行任务 [P]，继续完成成功项并报告失败项
   - 提供清晰的错误信息与调试上下文
   - 若无法继续实施，建议下一步
   - **重要**：完成任务后务必在 tasks 文件中将其标记为 [X]

9. 完成验证：
   - 确认所有必需任务已完成
   - 检查实现是否符合原始规范
   - 验证测试通过且覆盖率满足要求
   - 确认实现遵循技术计划
   - 输出最终状态与完成总结

注意：该命令假定 tasks.md 中已有完整任务拆分。若任务不完整或缺失，建议先运行 `/speckit.tasks` 重新生成任务列表。
