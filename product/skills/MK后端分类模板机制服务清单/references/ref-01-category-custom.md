# ref-01 — 分类（Category）自定义接口与端点

> 分类实体的业务层扩展：`I{Module}CategoryApi` 自定义方法 + `{Module}CategoryController` 自定义端点。
> 框架标准端点（add/get/update/delete/tree）不在此列出。

**核心规则**
- **search 端点需手动声明 `@PostMapping`**：框架 `ICategoryApi#search` 缺少该注解，Controller 必须显式添加，否则路由不生效
- **list 端点权限在代码内判断**：根据用户是否持有 ADMIN/CATEGORY_ADMIN 角色动态添加 AUTH_QUERYFILTER，非注解控制
- **copy 操作含子分类和模板**：递归复制，耗时较长，需考虑事务边界
- **loadParentCategoryVO 返回跨模块类型**：`LbpmFormCategoryVO` 来自 lbpm-form-api，需引入依赖（见 ref-00）

---

## API 自定义方法

`I{Module}CategoryApi extends ICategoryApi<{Module}CategoryVO>`

| # | 方法签名 | 说明 |
|---|---------|------|
| 1 | `Optional<LbpmFormCategoryVO> loadParentCategoryVO(IdVO idVO)` | 获取分类父节点（递归构建 LbpmFormCategoryVO 树） |
| 2 | `{Module}CategoryVO copy({Module}CategoryVO id)` | 复制分类（含子分类和模板） |
| 3 | `boolean checkHasTemplate(IdVO id)` | 检查分类下是否有模板 |
| 4 | `boolean checkUserAuth(String categoryId, AuthType authType)` | 校验当前用户对分类的权限 |
| 5 | `String getParentCategoryId(String categoryId)` | 获取父分类 ID |
| 6 | `List<{Module}CategoryVO> findByFdIds(List<String> fdIds)` | 按 ID 批量查询分类 |
| 7 | `Optional<{Module}CategoryVO> findRootCategoryByFdName(String fdName)` | 按名称查根分类 |
| 8 | `CommonTreeNodeVO loadParentNode(IdVO categoryIdVO)` | 加载分类祖先链（面包屑） |
| 9 | `IdVO ekpMigrate({Module}CategoryVO vo)` | EKP 迁移（按名称查重，存在则返回 ID，不存在则新增） |
| 10 | `List<{Module}CategoryVO> getCategoryFullPath(String categoryId)` | 获取分类完整路径（VO 列表） |

> 另有内部接口 `ICategoryApi`（业务自定义，非框架同名接口）：
> - `void fillParentCategoryVO({Module}CategoryVO, LbpmFormCategoryVO)` — 递归填充分类父节点信息

---

## Controller 自定义端点

**Controller**: `{Module}CategoryController`
**基路径**: `/data/{module-name}/{moduleName}Category`
**类级权限**:
- `@ValidateParams`: `ADMIN_ROLE_KEY` = `ROLE_{MODULE}_ADMIN`
- `@AuthValidators(OPT_OR)`: roleValidator(`ROLE_{MODULE}_SETTING`)

| # | HTTP | 路径 | 权限注解 | 说明 |
|---|------|------|---------|------|
| 1 | POST | `/search` | OPT_OR: roleValidator(DEFAULT) | 关键词搜索分类。**需手动声明 `@PostMapping("search")`** |
| 2 | POST | `/list` | 无注解（代码内判断 ADMIN/CATEGORY_ADMIN 后加 AUTH_QUERYFILTER） | 分页查询分类 |
| 3 | POST | `/copy` | OPT_OR: roleValidator(ADMIN, CATEGORY_ADMIN) \| CategoryEditorValidator(fdId, ONLY_CURRENT=true) | 复制分类（含子分类和模板），带 @OperLog |
| 4 | POST | `/checkHasTemplate` | OPT_OR: roleValidator(SETTING) \| CategoryReaderValidator(parentId) | 检查分类下是否有模板 |
| 5 | POST | `/findCategoryTree` | OPT_OR: roleValidator(DEFAULT) | 带权限过滤的分类树，注入 CategoryReaderFilter |
| 6 | POST | `/findCategoryCreateTree` | roleValidator(DEFAULT) | 过滤空分类的创建树，注入 TemplateCategoryFilter |
| 7 | POST | `/loadParentCategoryVO` | 无权限注解 | 查询分类父节点（LbpmFormCategoryVO） |
| 8 | POST | `/loadParentNode` | 无权限注解 | 查询分类祖先链（面包屑），返回 CommonTreeNodeVO |
| 9 | POST | `/getCategoryFullPath` | roleValidator(DEFAULT) | 获取分类完整路径（List<CategoryVO>） |

**易踩坑**
- `/search` 若不手动加 `@PostMapping("search")`，框架继承的接口方法没有路由，调用会 404
- `/list` 的权限不走注解，漏看代码逻辑容易误判为无鉴权接口
