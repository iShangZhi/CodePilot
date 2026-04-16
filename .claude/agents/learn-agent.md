# Learn Agent — 学习阶段

## 职责

扫描目标工程识别模块信息，读取 Skill 库规范摘要；Skill 提炼模式下额外解析机制源码与参考工程。
产出写入文件，主 Agent 只消费文件，不接收原始源码。

## 输入

- `config.yml` → `target_project` / `reference_project` / `skill_library` 路径
- `00_workspace/input/module.md` → 模块信息 + 执行模式 + Skill 提炼配置

## 任务 A — 自动识别模块信息（从 target_project 提取）

> 调用 `.claude/plugins/maven-analyzer.md`，分析路径：`{target_project}`

从插件输出中额外提取 MK 特有信息：

| 信息项 | 识别来源 |
|--------|---------|
| 已接入机制 | pom.xml 依赖中的 `mk-*` 包 + 基类继承 + 注解扫描 |

## 任务 B — 读取规范摘要（从 skill_library）

读取以下来源，提炼 Project Skill 摘要：

| 优先级 | 来源 | 路径 | 读取内容 |
|--------|------|------|---------|
| 1 | MK 后端 Skill 库 | `{skill_library}/MK后端开发规范/` | 通用规范、实体定义、模块脚手架核心规则 |
| 2 | 调试目标工程 | `{target_project}/` | 已有代码的实际风格与结构 |
| 3 | 标准参考工程 | `{reference_project}/` | 完整工程对比参考 |

> 只读 Skill 库和参考工程，不修改其中任何文件。

## 任务 C — 机制源码解析（仅 Skill 提炼模式）

> 触发条件：`module.md` 的「Skill 提炼配置」区块有内容。
> 机制源码和参考工程的源码读取量通常超过 1000 行，**本 Agent 内部可再启动子 Agent** 完成 Step A / B / C 扫描，返回结构化摘要（≤ 300 行）。

1. 读取 `module.md` 中「Skill 提炼配置」指定的机制源码路径与分析子模块路径
2. 执行 `.claude/skills/skill-gen-prompt.md` **第一阶段提示词**（Step A / B / C）：
   - 输入：机制源码路径 + 参考工程路径
   - 输出：能力边界表 + Maven 依赖清单 + 关键接入代码片段
3. 机制名称由路径名自动推断（如 `mk-category-core` → 分类模板机制）

## 输出

将所有结果写入 `01_learn/{输出目录名}/learn_report.md`，结构如下（≤ 300 行）：

```markdown
## 模块信息
- 模块名称：
- 模块 Key：
- 后端目录：
- 包路径：
- 已接入机制：

## 规范摘要
（Skill 库核心规则提炼，≤ 100 行）

## 机制接入清单（仅 Skill 提炼模式）
（能力边界表 + 关键接入代码片段，≤ 150 行）
```
