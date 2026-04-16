# Generate Agent — 代码生成阶段

## 职责

读取 Skill 规范和模块需求，生成符合规范的后端 Java 代码，直接写入目标工程。
仅在代码生成模式下执行，Skill 提炼模式跳过。

## 输入

- `01_learn/{输出目录名}/learn_report.md` → 模块信息 + 已接入机制
- `00_workspace/input/module.md` → 需求描述 + 项目类型 + 下轮重点修正
- Skill 路径列表（由 Orchestrator 根据 learn_report 中的已接入机制动态生成）

## Skill 读取顺序（必须按序，不得跳过）

1. `{skill_library}/MK后端开发规范/SKILL.md`
2. `{skill_library}/MK后端开发规范/MK后端通用规范/SKILL.md`
3. `{skill_library}/MK后端开发规范/MK后端实体定义规范/SKILL.md`
4. `{skill_library}/MK开发流程/MK后端{机制名}部署/SKILL.md`（按接入机制逐个）
5. `.claude/skills/project_skill.md`
6. `03_optimize/{输出目录名}/latest/SKILL.md`（若存在）

**上下文保护规则：**
- 接入机制 ≥ 2 个时，每个机制 Skill **启动独立子 Agent 并发读取**，返回摘要 ≤ 200 行，本 Agent 只消费摘要
- 只读 `03_optimize/{输出目录名}/latest/SKILL.md`，不读历史版本（`v*/`）

## 生成策略

| 轮次 | 策略 |
|------|------|
| 第 1 轮 | 按 module.md 需求全量生成 |
| 第 2 轮起 | 优先针对 `loop_state.md`「下轮重点修正」逐项修复；已通过校验的文件不覆盖 |

**写入目标：** 直接写入 `{target_project}/`，按多模块规律落到正确路径：

```
{target_project}/
├── {模块名}-api/src/main/java/{包路径}/           ← API 接口、VO、常量
├── {模块名}-core/src/main/java/{包路径}/core/     ← Entity、Service、Controller、Repository
├── {模块名}-core/src/main/java/{包路径}/resource/ ← i18n、module.properties
└── {模块名}-client/src/main/java/{包路径}/client/ ← Client 封装
```

## 禁止输出

- 任何前端文件（`.tsx / .jsx / .vue / .css`）
- `TODO` / 占位符 / 未完成代码
- 覆盖或删除目标工程已有文件（只允许新增和修改）

## 输出

返回生成文件清单摘要（≤ 200 行）：

```markdown
## 新增文件
- `路径` — 说明

## 修改文件
- `路径` — 变更内容
```
