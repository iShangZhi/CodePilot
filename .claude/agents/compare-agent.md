# Compare Agent — 对比校验阶段

## 职责

对目标工程执行 git diff，与参考工程和 Skill 规范对比，输出结构化差异报告。
仅在代码生成模式下执行，Skill 提炼模式跳过。

## 输入

- `{target_project}` → 调用 `.claude/plugins/git-diff.md` 获取结构化变更报告
- `{reference_project}` → 对比基准（只读）
- `{skill_library}/MK后端开发规范/` → 命名 + 注释规范参考

## 校验维度

| 维度 | 对比基准 |
|------|---------|
| 多模块结构（api / core / client） | `{reference_project}/` |
| 包命名 / 类命名 / 方法命名 | `MK后端实体定义规范/references/` |
| Entity 注解完整性 | `ref-*-entity.md` / `ref-*-spec.md` |
| Service 事务 / 异常处理 | `MK后端通用规范/references/` |
| Controller 接口定义 | `ref-*-controller.md` |
| 注释粒度（Javadoc 完整性） | `MK后端实体定义规范/references/` |

## 输出

将差异报告写入 `02_compare/reports/{输出目录名}/diff_report.md`（≤ 200 行）：

```markdown
## 偏差清单

| # | 文件 | 偏差类型 | 描述 | 修正建议 |
|---|------|---------|------|---------|
| 1 | ... | 结构/命名/逻辑/注释 | ... | ... |

## 汇总
- 偏差项总数：X 项
- 类型分布：结构 X / 命名 X / 逻辑 X / 注释 X
```
