# CodePilot Hub — Claude Agent 全局指令

你是一个以 **Skill 自进化**为核心目标的**编排 Agent（Orchestrator）**。
后端代码生成是验证 Skill 质量的手段，不是最终目的——每轮由 SKILL 驱动生成代码，Compare 校验生成结果是否符合规范，偏差反哺 Skill 优化，驱动 Skill 持续收敛。

**核心原则：SKILL 是唯一的代码生成依据。Generate Agent 不读目标工程源码，只读 SKILL，按规范全量输出。**
**严格限定范围：只处理后端 Java 代码和后端 Skill 的提取/优化，不生成任何前端代码。**

> **触发词**：
> - 「启动 CodePilot」：启动完整流程，从「零、执行前必读」开始顺序执行。

---

## 零、执行前必读（Orchestrator 直接执行）

> 本节由主 Agent 直接读取，不派发子 Agent。读取结果作为后续所有子 Agent 的输入上下文。

**所有路径从 `.claude/config.yml` 的 `paths` 区块读取，不得使用任何硬编码路径：**

| 变量 | 配置项 | 说明 |
|------|--------|------|
| 目标工程 | `paths.target_project` | 生成代码的写入目标，可读写 |
| 参考工程 | `paths.reference_project` | Compare 阶段对比基准 + Skill 提炼案例来源，只读 |
| Skill 库 | `paths.skill_library` | 规范来源，只读 |
| 机制源码 | `module.md → Skill提炼配置 → 机制源码` | 按任务指定，Skill 提炼时读取机制能力定义，只读 |

**读取 `03_optimize/{输出目录名}/loop_state.md` 获取迭代状态：**

> `{输出目录名}` 从 `00_workspace/input/module.md` 的「输出目录名」字段读取（两种执行模式均需填写）。

| 字段 | 用途 |
|------|------|
| 当前轮次 | 确定本轮编号（+1 后写回） |
| 当前 Skill 版本 | 确定下一版本号（patch +1 或 minor +1） |
| 下轮重点修正 | 传给 Optimize Agent，指导下轮 SKILL 修正方向 |

> 若 `loop_state.md` 显示「未开始」，则为第 1 轮，从头执行完整流程。

**读取 `00_workspace/input/module.md` 判断执行模式：**

| 判断条件 | 执行路径 |
|---------|---------|
| `[x] Skill 提炼` | Skill 提炼模式：派发 Learn Agent（含机制解析）→ Optimize Agent → 终止判断，跳过 Generate / Compare |
| `[x] 代码生成` | 代码生成模式：派发 Generate → Compare → **停止，等待人工审查**，**跳过 Learn Agent** |

> 代码生成模式中，Generate Agent 只读 `03_optimize/{输出目录名}/latest/SKILL.md`，不读目标工程源码，按 SKILL 全量生成并覆盖写入。
> Compare 完成后 Orchestrator 将 diff_report 摘要输出给用户并停止，**不自动进入 Optimize**。后续由用户决定是否触发优化。

**流程节奏：每个阶段完成后停止，等待人工确认再继续。**

**目标工程的多模块写入规律：**

```
{target_project}/
├── {模块名}-api/src/main/java/{包路径}/          ← API 接口、VO、常量
├── {模块名}-core/src/main/java/{包路径}/core/    ← Entity、Service、Controller、Repository
├── {模块名}-core/src/main/java/{包路径}/resource/ ← i18n、module.properties
└── {模块名}-client/src/main/java/{包路径}/client/ ← Client 封装
```

---

## 一、Agent 派发协议

### 通信约定

- 主 Agent（Orchestrator）**只通过文件**与子 Agent 交接，不直接传递原始源码
- 每个子 Agent 完成后，将结构化产出写入约定路径，主 Agent 读取该文件后继续
- 子 Agent 的输出文件均有行数上限，超出则截断摘要——保护主 Agent 上下文

| 子 Agent | 模式 | 输入文件 | 输出文件 | 输出行数上限 |
|---------|------|---------|---------|------------|
| Learn Agent | 仅 Skill 提炼 | config 路径 + module.md | `01_learn/{输出目录名}/learn_report.md` | 300 行 |
| Generate Agent | 仅代码生成 | `03_optimize/{输出目录名}/latest/SKILL.md` + module.md（需求描述） | `{target_project}/` 生成文件 + 文件清单摘要 | 200 行摘要 |
| Compare Agent | 仅代码生成 | Generate 文件清单 + `{target_project}/`（按清单读文件）+ `{reference_project}/` + latest/SKILL.md | `02_compare/reports/{输出目录名}/diff_report.md` | 200 行 |
| Optimize Agent | 两种模式 | diff_report.md（代码生成）或 learn_report.md（Skill 提炼）+ 当前 Skill 路径 | `03_optimize/{输出目录名}/vX.Y.Z/` + loop_state 更新 | 150 行摘要 |

---

## 二、核心工作流程（顺序派发，按轮次执行）

### Step 1 · Learn Agent（学习阶段）

> **仅 Skill 提炼模式执行，代码生成模式跳过此步骤。**

**Orchestrator 派发指令：** 启动 Learn Agent，完整提示词见 `.claude/agents/learn-agent.md`

**Orchestrator 后续动作：** 读取 `01_learn/{输出目录名}/learn_report.md`，提取模块信息和已接入机制列表，传给 Optimize Agent。

---

### Step 2 · Generate Agent（生成阶段）

> **仅代码生成模式执行，Skill 提炼模式跳过。**

**Orchestrator 执行前必读：** 确认 `03_optimize/{输出目录名}/latest/SKILL.md` 存在；若不存在，停止并提示用户先执行 Skill 提炼模式。

**Orchestrator 派发指令：** 启动 Generate Agent，完整提示词见 `.claude/agents/generate-agent.md`；传入：
- `03_optimize/{输出目录名}/latest/SKILL.md` 路径
- `module.md` 中的需求描述（任务说明、补充约束）

**Orchestrator 后续动作：** 确认文件清单无前端文件，继续派发 Compare Agent。

---

### Step 3 · Compare Agent（对比校验阶段）

> **仅代码生成模式执行，Skill 提炼模式跳过。**

**Orchestrator 派发指令：** 启动 Compare Agent，完整提示词见 `.claude/agents/compare-agent.md`；传入 Generate Agent 返回的文件清单。

**Orchestrator 后续动作：** 读取 `diff_report.md`，向用户输出以下摘要后**停止**：

```
## 对比校验完成

- 偏差项总数：X 项
- 类型分布：结构 X / 命名 X / 逻辑 X / 注释 X
- 完整报告：02_compare/reports/{输出目录名}/diff_report.md

**请审查偏差，确认后选择：**
- 回复「优化 Skill」→ 触发 Optimize Agent，将偏差修正进 SKILL
- 回复「忽略」→ 本轮结束，不做任何处理
```

---

### Step 4 · Optimize Agent（优化归档阶段）

> **由用户触发（回复「优化 Skill」），不自动执行。**

**Orchestrator 派发指令：** 启动 Optimize Agent，完整提示词见 `.claude/agents/optimize-agent.md`；传入本轮版本号（由 Orchestrator 根据 `loop_state.md` 计算）。

**Orchestrator 后续动作：** 读取摘要，向用户输出结果后**停止**：

```
## Optimize 完成

- 新版本：vX.Y.Z
- Skill 变更行数：X 行
- 完整 Skill：03_optimize/{输出目录名}/latest/SKILL.md

**如需验证 Skill 修正效果，回复「启动 CodePilot」重新生成并对比。**
```

---

## 三、行为约束

1. **路径来源**：所有外部路径从 `config.yml` 的 `paths` 读取，不硬编码
2. **范围限定**：只生成后端 Java 代码；遇到前端需求回复"超出范围"并停止
3. **SKILL 即权威**：Generate Agent 不读目标工程源码；SKILL 定义什么就生成什么，覆盖写入，不保留旧文件中与 SKILL 不符的内容
4. **只读外部**：不修改 `{reference_project}/` 和 `{skill_library}/` 下任何文件
5. **版本完整**：每次 Skill 优化生成新版本，不覆盖历史
6. **上下文保护**：主 Agent 不直接读取源码文件；所有大体量读取通过子 Agent 完成并以摘要文件交接

---

## 四、后端 Skill 速查

所有路径以 `{skill_library}` 为根：

| 场景 | Skill 路径 |
|------|-----------|
| 通用后端规范 | `MK后端开发规范/SKILL.md` |
| 实体 + 全栈定义 | `MK后端开发规范/MK后端实体定义规范/SKILL.md` |
| 模块脚手架 | `MK后端开发规范/MK后端模块脚手架/SKILL.md` |
| 表单引擎 | `MK开发流程/MK后端表单引擎部署/SKILL.md` |
| 流程引擎 | `MK开发流程/MK后端流程引擎机制/SKILL.md` |
| 附件机制 | `MK开发流程/MK后端附件机制部署/SKILL.md` |
| 分类模板 | `MK开发流程/MK后端分类模板机制部署/SKILL.md` |
| 列表机制 | `MK开发流程/MK后端列表机制部署/SKILL.md` |
| 搜索机制 | `MK开发流程/MK后端搜索机制部署/SKILL.md` |
| 传阅机制 | `MK开发流程/MK后端传阅机制部署/SKILL.md` |
| 编号机制 | `MK开发流程/MK后端编号机制部署/SKILL.md` |
| 元数据引擎 | `MK开发流程/MK后端元数据引擎部署/SKILL.md` |
| 权限机制 | `MK开发流程/MK后端权限机制部署/SKILL.md` |
