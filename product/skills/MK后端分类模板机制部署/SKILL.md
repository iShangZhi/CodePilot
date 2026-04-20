---
name: MK后端分类模板机制部署
description: 用于MK业务模块部署分类模版机制，实现独立分类管理、模板与分类关联、主文档按分类归属等功能。
---

# MK后端分类模板机制部署

## 一、实体类型速查

部署前先判断各实体类型，再按对应规范生成代码。

| 维度 | 主文档型（Main） | 分类型（Category） | 模板型（Template） |
|------|---------------|-----------------|-----------------|
| 识别特征 | 有标题/内容、流程审批、创建人权限控制 | 父子层级、树形展示、节点可启用/禁用 | 可编辑内容体、被引用/实例化、有修改人记录 |
| 典型场景 | 新闻、公告、申请单、报告 | 新闻分类、知识分类、菜单树 | 流程模板、打印模板、消息通知模板 |
| Entity 核心接口 | `FdSubject` + 时间/人员系列 | `CategoryEntity<E, Perm>`（内置 fdName/fdDesc/fdOrder/fdParent/fdHierarchyId）；可选实现 `FdIsTransferData`（数据迁移标记） | `FdName4Language` + `FdOrder` + 时间/人员系列；可选实现 `PermissionEntity<Perm>`（模板级权限控制） |
| 关联字段 | `fdTemplate`（`@ManyToOne`） | 无（自身即分类） | `fdCategory`（`@ManyToOne`，**字段名固定**） |
| VO 基类 | `AbstractVO` | `AbstractCategoryVO`（已含 fdName/fdDesc/fdOrder/fdParent/fdHierarchyId/fdTreeLevel/showType） | `AbstractVO` |
| API 接口 | `IApi<VO>` | `ICategoryApi<VO>` | `IApi<VO>` |
| Controller 接口 | `PreloadController` + `CombineController` | `CategoryController`（需显式声明 `tree` + `search`） | `CombineController` |

**关键约束**：
- **分类型**：`fdPermissions` 需显式声明 `@OneToMany`；`fdOrder` 需在 `entityToVo`/`voToEntity` 中手动映射（框架不自动映射）；`ICategoryApi#search` 无 `@PostMapping`，必须在 Controller 中显式暴露；`beforeSaveOrUpdate` 中须强制设置 `fdReaderFlag = false`（关闭分类级阅读者标记，权限由 fdPermissions 独立控制）；可选添加 `@AuthFieldFilters` 注解实现创建者字段级权限过滤；可选实现 `FdIsTransferData` 接口支持数据迁移
- **模板型**：关联分类的字段名**必须**为 `fdCategory`，框架层级查询路径依赖此名称
- **主文档型**：通过 `fdTemplate → fdCategory` 两级关联归属分类，查询时使用 `fdTemplate.fdCategory.fdHierarchyId`

---

## 二、核心实体关系

以新闻管理模块为例：

| 角色 | 示例类名 | 说明 |
|------|---------|------|
| 分类实体 | `KmNewsCategory` | 管理分类树结构，实现 `CategoryEntity` 接口 |
| 分类权限实体 | `KmNewsCategoryPerm` | 分类行级权限控制，继承 `PermissionInfoEntity` |
| 模板实体 | `KmNewsTemplate` | 通过 `fdCategory` 字段关联分类 |
| 主文档实体 | `KmNewsMain` | 通过 `fdTemplate` 字段归属到模板 |

---

## 三、部署流程

### 步骤1：收集模块信息

优先从上下文自主获取，无法获取时向用户确认：

| 信息项 | 示例 |
|--------|------|
| 模块名称（中文） | 新闻管理 |
| 模块Key | `km-news` |
| 模块包名前缀 | `km` |
| 模块包名后缀 | `news` |
| 模板实体完整类名 | `KmNewsTemplate` |
| 主文档实体完整类名 | `KmNewsMain` |

### 步骤2：确认部署信息

展示如下确认信息，等待用户确认后继续：

```
分类模版模式部署方案：
- 目标模块：{模块Key}（{模块名称}）
- 分类实体：{Module}Category
- 分类权限实体：{Module}CategoryPerm
- 模板实体：{Module}Template
- 主文档实体：{Module}Main

确认是否按此方案部署？(是/否)
```

### 步骤3：添加 Maven 依赖

在 `mk-{模块Key}-api/pom.xml` 和 `mk-{模块Key}-core/pom.xml` 中添加分类机制依赖：

```xml
<dependency>
    <groupId>${mk.group-id}</groupId>
    <artifactId>mk-sys-category-api</artifactId>
</dependency>
```

### 步骤4：执行部署

按以下顺序依次完成三个机制的部署：

| 机制 | 参考文档 | 依赖 | 说明 |
|------|---------|------|------|
| 包导入规范 | [ref-00-bundle-spec.md](references/ref-00-bundle-spec.md) | — | **所有 ref 均须遵守**；集中声明跨模块包导入 |
| 分类 | [ref-01-category-spec.md](references/ref-01-category-spec.md) | 无 | Entity、Perm、VO、API、Repository、Service、Controller、i18n |
| 模板基础 | [ref-02-template-base-spec.md](references/ref-02-template-base-spec.md) | 分类 | 枚举、Entity、VO、API、Repository、Service、Controller（CRUD + publish）、i18n |
| 模板树查询 | [ref-03-template-tree-spec.md](references/ref-03-template-tree-spec.md) | 模板基础 | **步骤0**：`TemplateTreeFilterType` 枚举（constant 包）→ **步骤1**：树节点 VO（`TemplateTreeRequestVO` 依赖步骤0枚举）→ **步骤2**：SearchService → **步骤3**：Controller 树端点（getMgmtTree / getRuntimeTree / treeSearch） |
| 主文档 | [ref-04-main-spec.md](references/ref-04-main-spec.md) | 模板 | Entity（添加 fdTemplate）、VO、API、Repository、Service、Controller、i18n |

---

## 四、部署验证清单

- [ ] Maven 依赖已添加
- [ ] `{Module}Category` Entity 已创建
- [ ] `{Module}CategoryPerm` Entity 已创建
- [ ] `{Module}Template` Entity 已添加 `fdCategory` 字段
- [ ] `{Module}Main` Entity 已添加 `fdTemplate` 字段
- [ ] 分类 VO、API、Repository、Service、Controller 已创建
- [ ] 模板 VO、API、Repository、Service、Controller 已创建
- [ ] 树节点 VO（TreeRequestVO / TreeNodeVO）已创建
- [ ] SearchService 已创建，getMgmtTree / getRuntimeTree / treeSearch 端点可用
- [ ] 主文档 VO、API、Repository、Service、Controller 已创建
- [ ] 国际化配置已添加
- [ ] 分类树接口返回正常
- [ ] 模板与分类关联正常
- [ ] 主文档按分类查询正常

---

## 五、代码注释规范

生成代码时必须遵守以下注释要求：

| 位置 | 要求 |
|------|------|
| **接口方法**（API / Repository） | 必须写 Javadoc：一句话说明用途、`@param` 描述每个参数含义、`@return` 描述返回值及其结构 |
| **Service 公开方法** | 必须写 Javadoc：说明业务语义、状态流转（如有）、与其他方法的依赖关系 |
| **Service 私有辅助方法** | 必须写 `/** ... */` 注释：说明职责及关键逻辑，复杂逻辑在方法体内用行内注释补充 |
| **Entity / VO 字段** | 非自解释字段必须用 `/** ... */` 注明：含义、约束（如"字段名必须为 fdCategory"）、与其他字段的关联 |
| **Controller 端点方法** | 必须写 Javadoc：HTTP 路径、业务语义、权限要求、`@param`/`@return` |
| **类级注释** | 所有新建类必须有类级 Javadoc：说明职责、关键约束（如 Feign 路径、循环依赖处理方式等） |

> **行内注释原则**：注释说明"为什么"而非"做什么"；框架约定（如排除字段的理由、排序方向的原因）必须注明，否则后续维护者无从判断是否可以修改。

---

## 六、占位符说明

| 占位符 | 说明 | 示例 |
|--------|------|------|
| `{Module}` | 模块实体类名前缀，大驼峰 | `KmNews` |
| `{module}` | 模块实体名，小驼峰 | `kmNews` |
| `{模块Key}` | 模块标识，小写+连字符 | `km-news` |
| `{modulePrefix}` | 模块包名前缀 | `km` |
| `{moduleSuffix}` | 模块包名后缀 | `news` |
