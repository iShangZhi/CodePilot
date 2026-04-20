# ref-00 — 跨模块类型声明与 Maven 依赖

> 调用分类模板机制服务时，方法签名中涉及以下来自其他模块的类型，需在调用方模块引入对应依赖。

---

## 跨模块类型清单

| 类型 | 来源模块（artifactId） | 用途 |
|------|----------------------|------|
| `LbpmFormCategoryVO` | `lbpm-form-api` | 分类父节点递归结构（loadParentCategoryVO 返回值） |
| `CommonTreeNodeVO` | `sys-common-api` | 通用树节点（面包屑、祖先链） |
| `IdVO` | `sys-common-api` | 单 ID 入参包装 |
| `IdsDTO` | `sys-common-api` | 批量 ID 入参包装 |
| `IdNameProperty` | `sys-common-api` | ID + Name 键值对 |
| `QueryResult<T>` | `sys-common-api` | 分页查询结果包装 |
| `AuthType` | `sys-auth-api` | 权限类型枚举（READ / WRITE 等） |
| `QueryRequest` | `sys-query-api` | 通用查询条件对象 |
| `TreeParamDTO` | `sys-category-api` | 分类树查询参数 |
| `TemplateTreeNodeVO` | `{module}-api`（本模块） | 模板树节点（跨 Service 调用时注意） |
| `TemplateTreeRequestVO` | `{module}-api`（本模块） | 模板树查询请求 |

---

## Maven 依赖片段

在调用方模块的 `pom.xml` 中按需引入：

```xml
<!-- 通用基础类型：IdVO / IdsDTO / IdNameProperty / CommonTreeNodeVO / QueryResult -->
<dependency>
    <groupId>com.landray</groupId>
    <artifactId>sys-common-api</artifactId>
</dependency>

<!-- 权限类型：AuthType -->
<dependency>
    <groupId>com.landray</groupId>
    <artifactId>sys-auth-api</artifactId>
</dependency>

<!-- 通用查询：QueryRequest -->
<dependency>
    <groupId>com.landray</groupId>
    <artifactId>sys-query-api</artifactId>
</dependency>

<!-- 分类树查询参数：TreeParamDTO -->
<dependency>
    <groupId>com.landray</groupId>
    <artifactId>sys-category-api</artifactId>
</dependency>

<!-- LBPM 表单分类 VO（仅在 loadParentCategoryVO 场景下需要） -->
<dependency>
    <groupId>com.landray</groupId>
    <artifactId>lbpm-form-api</artifactId>
</dependency>
```

> 以上依赖通常已通过 BOM 管理版本，无需指定 `<version>`。
