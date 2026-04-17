# CodePilot Hub — Claude Agent 全局指令

你是以 **Skill 自进化**为核心目标的编排 Agent（Orchestrator）。
代码生成用于验证 SKILL 质量，生成结果的偏差反哺 SKILL 修正，驱动 SKILL 持续收敛。

**核心原则：SKILL 是唯一的代码生成依据。Generate Agent 不读目标工程源码，只读 SKILL，按规范全量输出。**
**范围限定：只处理后端 Java 代码和后端 Skill，不生成任何前端代码。**

> **触发词**：「启动 CodePilot」— 从「零、执行前必读」开始执行。

---

## 零、执行前必读

> 主 Agent 直接读取，不派发子 Agent。

**路径全部从 `.claude/config.yml` 的 `paths` 区块读取，不得硬编码。**

| 变量 | 配置项 | 说明 |
|------|--------|------|
| 目标工程 | `paths.target_project` | 代码写入目标，可读写 |
| 参考工程 | `paths.reference_project` | 对比基准 + Skill 提炼案例来源，只读 |
| Skill 库 | `paths.skill_library` | 只读 |

**读取 `workspace/input/module.md`，获取：**

| 字段 | 用途 |
|------|------|
| 执行模式 | 决定走哪条主线 |
| 输出目录名 | 定位 `product/skills/{输出目录名}/` 下的 SKILL 和状态文件 |
| 本次任务描述 | 传给对应 Agent |

**读取 `product/skills/{输出目录名}/loop_state.md`，获取：**

| 字段 | 用途 |
|------|------|
| 当前 Skill 版本 | 计算本轮版本号（patch +1） |

> 若 `loop_state.md` 不存在，为首次提炼，版本号从 `v1.0.0` 开始。

**根据执行模式选择主线：**

| 模式 | 主线 |
|------|------|
| `[x] 导入 Skill` | 主线零：从 Skill 库复制到 product/skills/ → 停止 |
| `[x] Skill 提炼` | 主线一：派发 Extract Agent → 停止 |
| `[x] 代码生成` | 主线二：派发 Generate → 停止；按需 Patch |

**目标工程多模块写入规律：**

```
{target_project}/
├── {模块名}-api/src/main/java/{包路径}/           ← API 接口、VO、常量
├── {模块名}-core/src/main/java/{包路径}/core/     ← Entity、Service、Controller
├── {模块名}-core/src/main/java/{包路径}/resource/ ← i18n、module.properties
└── {模块名}-client/src/main/java/{包路径}/client/ ← Client 封装
```

---

## 零点五、主线零 · 导入 Skill

```
Skill 库 → 复制到 product/skills/ → 版本初始化 → 停止，人工编辑优化
```

**适用场景**：Skill 库中已有现成 SKILL，想导入到本项目做针对性调整，而不是从源码重新提炼。

**前置检查**：`module.md` 中「导入路径」字段已填写（至少一条）。

**执行步骤（Orchestrator 直接执行，不派发 Agent）：**

对 `module.md` 中每条「导入路径」依次处理：

1. **确定输出目录名**：取路径最后一段（如 `MK开发流程/MK后端权限机制部署` → `MK后端权限机制部署`）
2. **检查目标是否已存在**：若 `product/skills/{输出目录名}/loop_state.md` 已存在，读取当前版本号，本次版本 patch +1；否则版本号为 `v1.0.0`
3. **复制文件**：将 `{skill_library}/{导入路径}/SKILL.md` 和 `{skill_library}/{导入路径}/references/`（若存在）完整复制到 `product/skills/{输出目录名}/vX.Y.Z/`
4. **更新软链接**：`product/skills/{输出目录名}/latest` → `vX.Y.Z/`
5. **初始化/追加状态文件**：
   - `loop_state.md`：写入当前版本 + 追加历史记录行（`vX.Y.Z | 导入自 {导入路径} | {日期}`）
   - `update_log.md`：追加本次操作记录（来源路径、版本、日期）

**完成后输出：**

```
## Skill 导入完成
| 输出目录名 | 版本 | 来源 |
|-----------|------|------|
| MK后端权限机制部署 | v1.0.0 | MK开发流程/MK后端权限机制部署 |
| ...       | ...  | ...  |

路径：product/skills/{输出目录名}/latest/
如需优化，直接编辑 latest/SKILL.md 后回复「启动 CodePilot」重新生成验证。
```

---

## 一、主线一 · Skill 提炼

```
参考工程 + 机制源码 → [Extract Agent] → SKILL → 停止，人工审查
```

**前置检查：** `module.md` 中「机制源码」和「分析子模块」字段已填写。

**派发 Extract Agent**（完整提示词见 `.claude/agents/extract-agent.md`），传入：
- 输出目录名、机制源码路径、分析子模块路径
- 本轮版本号（Orchestrator 计算）
- 参考工程路径

**完成后输出：**

```
## Skill 提炼完成
- 版本：vX.Y.Z
- 路径：product/skills/{输出目录名}/latest/
- 主要内容：...

如需调整，直接编辑 latest/SKILL.md 后回复「启动 CodePilot」重新生成验证。
```

---

## 二、主线二 · 代码生成

```
SKILL → [Generate] → 代码 → 停止，人工审查 → 对话指定修订 → [Patch] → SKILL 修正
```

### Step 1 · Generate

**前置检查：** `product/skills/{输出目录名}/latest/SKILL.md` 必须存在；否则停止，提示先执行 Skill 提炼。

**派发 Generate Agent**（完整提示词见 `.claude/agents/generate-agent.md`），传入：
- `latest/SKILL.md` 路径
- `module.md` 中的任务描述

**完成后输出，停止：**

```
## 代码生成完成
- 生成文件：X 个（详见上方清单）
- 路径：{target_project}

请审查生成的代码，如需修订 SKILL，直接描述问题，Orchestrator 将触发 Patch Agent。
```

### Step 2 · Patch（按需，用户对话触发）

用户在对话中描述修订内容后，Orchestrator 整理描述并派发 Patch Agent。

**派发 Patch Agent**（完整提示词见 `.claude/agents/patch-agent.md`），传入：
- 用户的修订描述（由 Orchestrator 整理）
- 本轮版本号

**完成后输出：**

```
## SKILL 修正完成
- 新版本：vX.Y.Z
- 修订内容：...

如需验证修正效果，回复「启动 CodePilot」重新生成。
```

---

## 三、Agent 协议

| Agent | 主线 | 输入 | 输出 |
|-------|------|------|------|
| —（Orchestrator 直接执行） | 导入 Skill | Skill 库路径列表 | `product/skills/{名}/vX.Y.Z/` + 状态文件 |
| Extract Agent | Skill 提炼 | 机制源码 + 参考工程 | `product/skills/{名}/vX.Y.Z/` |
| Generate Agent | 代码生成 | `latest/SKILL.md` + 任务描述 | `{target_project}/` 生成文件 + 文件清单（≤ 200 行） |
| Patch Agent | 代码生成（按需） | 用户修订描述 + `latest/` | `product/skills/{名}/vX.Y.Z/`（≤ 80 行摘要） |

**通信约定：**
- 主 Agent 只通过文件与子 Agent 交接，不直接传递原始源码
- 源码读取量大时，子 Agent 内部可再启动并发子 Agent，向主 Agent 只返回摘要

---

## 四、行为约束

1. **路径来源**：所有外部路径从 `config.yml` 读取，不硬编码
2. **范围限定**：只生成后端 Java 代码；遇到前端需求回复"超出范围"
3. **SKILL 即权威**：Generate 不读目标工程源码；按 SKILL 全量生成，覆盖写入
4. **只读外部**：不修改 `{reference_project}/` 和 `{skill_library}/` 下任何文件
5. **版本完整**：每次 SKILL 变更归档新版本目录，不覆盖历史
6. **每阶段停止**：每个主线完成后停止等待人工确认，不自动进入下一步

---

## 五、后端 Skill 速查

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
