---
name: project-skill
description: CodePilot Hub 项目特定后端规范，由 Learn Agent 自动生成和更新，记录从目标工程提取的多模块结构、实体层、Service、Controller 等项目级约定。
---

# Project Skill — CodePilot Hub（后端）

> 本文件由 Claude Agent 在 Learn 阶段自动生成，记录从目标后端工程中提取的项目特定规范。
> 仅包含后端相关内容；每次 Optimize 后更新，最新稳定版归档至 `03_optimize/latest/`。

---

## Skill 读取优先级

| 优先级 | 来源 | 路径（根路径从 config.yml `paths.skill_library` 读取） |
|--------|------|------|
| 1（最高） | MK 后端 Skill 库 | `{skill_library}/MK后端开发规范/` |
| 2 | 后端机制 Skill | `{skill_library}/MK开发流程/MK后端*/` |
| 3 | 本文件（项目特定） | `.claude/skills/project_skill.md` |
| 4 | 优化归档版本 | `03_optimize/latest/` |

---

## 后端 Skill 速查

| 场景 | Skill 路径（相对于 `.trae/skills/`） |
|------|-------------------------------------|
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

---

## 【待 Agent 填充 — Learn 阶段写入】

以下内容由 Claude Agent 读取 MK 后端 Skill 库（`{skill_library}`）+ 参考工程（`{reference_project}`）后自动填充：

1. **多模块 Maven 结构**：root / api / core / client / server 各模块职责与包路径规律
2. **实体层规范**：基类继承、字段注解（`@EKPColumn`等）、命名约定
3. **VO / API 接口定义规范**：接口继承结构、方法签名约定
4. **Service 规范**：事务边界、异常抛出方式、日志记录
5. **Controller 规范**：路由前缀规律、参数校验、统一返回格式
6. **Repository/Mapper 规范**：查询方法命名、分页参数
7. **i18n 规范**：`ApplicationResources.properties` 的 key 命名规则
8. **项目特定约束**：与 MK Skill 库通用规范不同的项目级特殊约定

---

## Skill 提炼规范（提炼新机制 Skill 时使用）

- **生成提示词模板**：[skill-gen-prompt.md](skill-gen-prompt.md)
- **命名与结构约定**：[references/ref-00-conventions.md](references/ref-00-conventions.md)
- **机制类型与清单**：[references/ref-01-catalog.md](references/ref-01-catalog.md)
