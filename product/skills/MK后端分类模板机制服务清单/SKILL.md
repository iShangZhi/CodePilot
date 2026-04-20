---
name: MK后端分类模板机制服务清单
description: 分类+模板机制业务层自定义 API 方法、Controller 端点（含权限注解）和 Service public 方法清单
---

# MK后端分类模板机制服务清单

> 本文档只列出**业务自定义**内容。框架标准端点（init/add/get/update/delete/deleteAll/list/meta/tree）已在「MK后端分类模板机制部署」SKILL 中说明，此处不重复。

---

## 一、服务概览

| 实体域 | API 自定义方法 | Controller 自定义端点 | ref 文件 |
|--------|-------------|-------------------|---------|
| 分类 Category | 10 个方法 | 9 个端点 | [ref-01-category-custom.md](references/ref-01-category-custom.md) |
| 模板 Template | 14 个方法 | 24 个端点 | [ref-02-template-custom.md](references/ref-02-template-custom.md) |

---

## 二、Service public 方法概览

| Service 类 | 方法数量 | ref 链接 |
|-----------|---------|---------|
| CategoryService | 4 个 | [ref-03-service-methods.md](references/ref-03-service-methods.md) |
| TemplateService | 11 个 | [ref-03-service-methods.md](references/ref-03-service-methods.md) |
| SearchService | 4 个 | [ref-03-service-methods.md](references/ref-03-service-methods.md) |

---

## 三、工具类概览

| 工具类 | 核心方法 | ref 链接 |
|--------|---------|---------|
| `{Module}CategoryUtil` | `swapHierarchyId` — 将 child 查询条件转换为 startsWith | [ref-04-utils.md](references/ref-04-utils.md) |

---

## 四、权限角色与校验器速查

### 4.1 核心角色常量

**常量类**: `{Module}Role`（位于 `core.config` 包）

| 常量 | 值 | 说明 | 继承角色 |
|------|-----|------|---------|
| `ROLE_{MODULE}_DEFAULT` | `"ROLE_{MODULE}_DEFAULT"` | 默认权限（所有用户基础权限） | 无 |
| `ROLE_{MODULE}_ADMIN` | `"ROLE_{MODULE}_ADMIN"` | 管理员（最高权限） | SETTING, DEFAULT, DATA_MANAGE_MAINTENANCE, SYS_TEMPLATE_MAINTENANCE, MAIN_CREATE, MAIN_READER, MAIN_UPDATE, MAIN_DELETE, CATEGORY_ADMIN, QUERY_EXPORT, MAIN_ARCHIVE, MAIN_SUBSIDE |
| `ROLE_{MODULE}_SETTING` | `"ROLE_{MODULE}_SETTING"` | 后台管理 | MENU_SYS_APPLICATION_ROLE |
| `ROLE_{MODULE}_CATEGORY_ADMIN` | `"ROLE_{MODULE}_CATEGORY_ADMIN"` | 类别维护人员 | 无 |
| `ROLE_{MODULE}_ADVANCED_SETTING` | `"ROLE_{MODULE}_ADVANCED_SETTING"` | 高级设置 | 无 |

### 4.2 文档操作角色

| 常量 | 说明 |
|------|------|
| `ROLE_{MODULE}_MAIN_CREATE` | 创建文档 |
| `ROLE_{MODULE}_MAIN_READER` | 阅读文档 |
| `ROLE_{MODULE}_MAIN_UPDATE` | 编辑文档 |
| `ROLE_{MODULE}_MAIN_DELETE` | 删除文档 |
| `ROLE_{MODULE}_MAIN_ARCHIVE` | 归档权限 |
| `ROLE_{MODULE}_MAIN_SUBSIDE` | 沉淀权限 |

### 4.3 功能性角色

| 常量 | 说明 |
|------|------|
| `ROLE_{MODULE}_QUERY_EXPORT` | 导出查询列表数据 |
| `ROLE_{MODULE}_TEMPLATE_PUBLISH` | 模板发布权限 |
| `ROLE_{MODULE}_DATA_MANAGE_SETTING` | 数据权限管理配置（继承 SETTING） |
| `ROLE_{MODULE}_DATA_MANAGE_MAINTENANCE` | 数据权限管理维护（继承 DATA_MANAGE_SETTING） |
| `ROLE_{MODULE}_RIGHT_CHANGE_MANAGER` | 维护所有权限变更记录 |
| `ROLE_{MODULE}_RIGHT_CHANGE_CREATOR` | 添加并维护自己创建的权限变更记录 |
| `ROLE_{MODULE}_SYS_TEMPLATE_SETTING` | 通用模板后台管理 |
| `ROLE_{MODULE}_SYS_TEMPLATE_MAINTENANCE` | 通用模板维护者（继承 SYS_TEMPLATE_SETTING） |

### 4.4 特殊常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `CREATOR` | `"creator"` | 创建者标识（非角色，用于 authFieldValidator） |

### 4.5 权限校验器

| 校验器 ID | 说明 | 参数 |
|-----------|------|------|
| `roleValidator` | 角色校验 | 角色常量字符串 |
| `{Module}CategoryEditorValidator` | 分类编辑权限校验 | `FD_ID` / `PARENT_ID` / `ONLY_CURRENT` |
| `{Module}CategoryReaderValidator` | 分类阅读权限校验 | `FD_ID` / `PARENT_ID` |
| `{Module}TemplateEditorValidator` | 模板编辑权限校验 | `CATEGORY_ID` / `TEMPLATE_ID` / `TEMPLATE_IDS` |
| `TemplatePublishAuthValidator` | 模板发布权限校验（开关） | 无 |
| `templatePrePublishAuthValidator` | 预发布开关校验 | `PUBLISH_STATUS` |
| `authFieldValidator` | 字段级权限校验 | `filterType`（CREATOR / SYS_EDITOR） |
| `categoryVisitorAuthValidator` | 分类可使用者校验 | `CATEGORY_ID` |
| `categoryMaintainerAuthValidator` | 分类可维护者校验 | `CATEGORY_ID` / `TEMPLATE_ID` |
