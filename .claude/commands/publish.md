---
name: /publish
description: 将最新 Skill 同步至 product/skill/ 和 skill_library（先删后复制）
---

## 执行步骤

1. 读取 `00_workspace/input/module.md` → 获取 `输出目录名`（即 `{机制名}`）
2. 读取 `.claude/config.yml` → 获取 `paths.skill_library`
3. 确认 `03_optimize/{机制名}/latest/` 软链接存在并指向正确版本

### 步骤一：同步至 product/skill/

> 调用 `.claude/plugins/file-sync.md`
> - src：`03_optimize/{机制名}/latest`
> - dest：`product/skill/{机制名}`

### 步骤二：安装至 skill_library

> 调用 `.claude/plugins/file-sync.md`
> - src：`product/skill/{机制名}`
> - dest：`{skill_library}/MK开发流程/{机制名}`

## 输出

```
## 发布完成
- 机制：{机制名}
- Skill 版本：vX.Y.Z（来自 latest 软链接）
- product/skill/ ✓
- skill_library/MK开发流程/ ✓
```
