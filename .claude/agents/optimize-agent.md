# Optimize Agent — 优化归档阶段

## 职责

根据 diff 报告优化 Skill，归档新版本，更新迭代状态，生成样例产物。

## 输入

- `02_compare/reports/{输出目录名}/diff_report.md` → 偏差清单（代码生成模式）
- `01_learn/{输出目录名}/learn_report.md` → 机制接入清单（Skill 提炼模式）
- `03_optimize/{输出目录名}/latest/` → 当前 Skill（只读参考）
- 本轮版本号（由 Orchestrator 根据 loop_state.md 计算后传入）

## 任务 A — 生成新版本 Skill

在 `03_optimize/{输出目录名}/vX.Y.Z/` 下生成完整 Skill 目录（不覆盖历史版本）：

```
03_optimize/{输出目录名}/vX.Y.Z/
├── SKILL.md                     # frontmatter(name+description) + 规范正文
└── references/
    ├── ref-00-bundle-spec.md    # 全局包导入规范（所有 ref 均须遵守）
    ├── ref-01-{能力}-spec.md    # 机制能力型（含 Entity 的全栈代码）
    └── ref-0N-{能力}-spec.md    # 独立能力（枚举/工具类/配置/i18n）
```

**Skill 内容生成依据：**
- 代码生成模式：对照 diff_report.md 偏差清单，修正上一版本 Skill 中对应规则
- Skill 提炼模式：基于 learn_report.md 机制接入清单，执行 `.claude/skills/skill-gen-prompt.md` 第二阶段提示词

## 任务 B — 更新软链接与状态文件

> 调用 `.claude/plugins/version-manager.md`，基础目录：`03_optimize/{输出目录名}`，新版本号：`{vX.Y.Z}`

插件负责：目录创建、latest 软链接、loop_state.md 写入。插件执行完后，额外执行（MK 专属，插件不处理）：
- 更新 `.claude/skills/project_skill.md`
- 追加 `03_optimize/{输出目录名}/update_log.md`：

```markdown
## vX.Y.Z — YYYY-MM-DD
MK Skill 库 commit：[git -C {skill_library} rev-parse --short HEAD]
变化来源：[ 规范库更新 / 代码对比优化 / 两者都有 ]
偏差点：...
修正内容：...
```

## 任务 C — 生成样例产物（product/sample/）

在 `product/sample/{模块名}-vX.Y.Z/` 下生成两个文件：

**README.md**
```markdown
## 模块信息
- 模块名称 / Key / 包路径
- 对应 Skill 版本：vX.Y.Z
- 生成日期：YYYY-MM-DD
- 接入机制：[列出已接入的机制]

## 完整代码位置
{target_project}

## 安装到 Skill 库
```bash
rm -rf "{skill_library}/MK开发流程/{模块名}后端部署" && \
cp -r "product/skill/{模块名}后端部署" "{skill_library}/MK开发流程/"
```
```

**files.md**
```markdown
## 新增文件
- `路径` — 说明

## 修改文件
- `路径` — 变更内容
```

## 输出

返回摘要（≤ 150 行）：

```markdown
## Optimize 完成
- 新版本：vX.Y.Z
- Skill 变更行数：X 行
- 下轮重点修正：
  1. ...
  2. ...
```
