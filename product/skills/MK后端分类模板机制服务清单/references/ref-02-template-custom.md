# ref-02 — 模板（Template）自定义接口与端点

> 模板实体的业务层扩展：`I{Module}TemplateApi` 自定义方法 + `{Module}TemplateController` 自定义端点。
> 框架标准端点（init/add/get/update/delete/deleteAll/list/meta）不在此列出。

**核心规则**
- **发布类操作均带 `@OperLog`**：publish / prePublish / batchPublish / copyTemplate / batchEnable / exportTemplate 均需记录操作日志
- **prePublish 使用 OPT_AND 组合校验**：需同时满足编辑权限和预发布开关，两个条件缺一不可
- **getMechAuthToken 权限最复杂**：OPT_AND 外层 + OPT_OR 内层双重组合，需仔细对照注解写法
- **batchUpdatePerm 为异步操作**：调用后需通过 `/getBatchUpdatePermProgress` 轮询进度

---

## API 自定义方法

`I{Module}TemplateApi extends IApi<{Module}TemplateVO>`

| # | 方法签名 | 说明 |
|---|---------|------|
| 1 | `{Module}TemplateVO publish({Module}TemplateVO templateVO)` | 发布模板（@PostMapping） |
| 2 | `boolean checkHasMain(IdVO id)` | 检查模板下是否有文档（@PostMapping） |
| 3 | `List<IdNameProperty> batchCheckHasMain(IdsDTO ids)` | 批量检查（@PostMapping） |
| 4 | `boolean checkUserAuth(String templateId, AuthType authType)` | 校验当前用户对模板的权限 |
| 5 | `void batchEnable({Module}BatchEnableVO batchEnableVO)` | 批量禁用/启用（@PostMapping） |
| 6 | `{Module}TemplateAvailableVO checkAvailable({Module}TemplateAvailableVO vo)` | 检查模板是否可用（@PostMapping） |
| 7 | `Boolean checkCodeUniqueness({Module}TemplateVO vo)` | 检查编码唯一性 |
| 8 | `void updateWrap({Module}TemplateVO vo)` | 更新并发布事件（@PostMapping） |
| 9 | `Map<String, String> getTemplateNames(List<String> fdIds)` | 批量获取模板名称 |
| 10 | `Response<?> batchPublish(IdsDTO ids)` | 批量发布 |
| 11 | `List<String> findCurCateHieraByReadTemplate(boolean isAdmin)` | 获取当前用户可读模板的分类层级 |
| 12 | `IdVO batchUpdatePerm({Module}TemplatePermVO permVO)` | 批量修改权限 |
| 13 | `Set<String> filterCategoryIdWithTemplate(List<String> categoryIds)` | 过滤出含模板的分类 ID |
| 14 | `QueryResult<IdNameProperty> getDocTemplateListByUser({Module}DocTemplateRequestVO vo)` | 查询用户相关有文档的模板（@PostMapping） |

---

## Controller 自定义端点

**Controller**: `{Module}TemplateController`
**基路径**: `/data/{module-name}/{moduleName}Template`
**类级权限**: `@AuthValidators(OPT_OR)`: roleValidator(SETTING) | roleValidator(ADMIN)

| # | HTTP | 路径 | 权限注解 | 说明 |
|---|------|------|---------|------|
| 1 | POST | `/publish` | @CompsiteValidators(OPT_OR): {roleValidator(CATEGORY_ADMIN, ADMIN)} \| {TemplatePublishAuthValidator AND TemplateEditorValidator(categoryId, templateId)} | 发布模板，带 @OperLog |
| 2 | POST | `/prePublish` | @CompsiteValidators(OPT_AND): {OPT_OR: roleValidator(CATEGORY_ADMIN, ADMIN) \| TemplateEditorValidator(categoryId, templateId)} AND {templatePrePublishAuthValidator(PUBLISH_STATUS)} | 预发布模板，带 @OperLog |
| 3 | POST | `/batchPublish` | @CompsiteValidators(OPT_OR): {OPT_OR: roleValidator(ADMIN) \| roleValidator(CATEGORY_ADMIN)} \| {OPT_AND: TemplatePublishAuthValidator AND TemplateEditorValidator(templateIds)} | 批量发布，带 @OperLog |
| 4 | POST | `/copyTemplate` | OPT_OR: roleValidator(CATEGORY_ADMIN, ADMIN) \| TemplateEditorValidator(categoryId) | 复制单个模板，带 @OperLog |
| 5 | POST | `/generateTableName` | 类级权限（无方法级注解） | 生成表单数据库表名 |
| 6 | POST | `/batchEnable` | OPT_OR: roleValidator(CATEGORY_ADMIN, ADMIN) \| TemplateEditorValidator(templateIds) | 批量禁用/启用，带 @OperLog |
| 7 | POST | `/getMgmtTree` | @CompsiteValidators(OPT_AND): {OPT_OR: roleValidator(SETTING) \| roleValidator(ADMIN)} AND {OPT_OR: roleValidator(ADMIN) \| roleValidator(CATEGORY_ADMIN) \| roleValidator(ADVANCED_SETTING) \| categoryVisitorAuthValidator(categoryId)} | 管理后台分类+模板树 |
| 8 | POST | `/treeSearch` | OPT_OR: roleValidator(DEFAULT) | 关键词搜索模板树 |
| 9 | POST | `/getRuntimeTree` | OPT_OR: roleValidator(DEFAULT) | 运行时分类+模板树 |
| 10 | POST | `/checkHasMain` | OPT_OR: roleValidator(DEFAULT) | 校验模板下是否有文档 |
| 11 | POST | `/batchCheckHasMain` | OPT_OR: roleValidator(DEFAULT) | 批量校验 |
| 12 | POST | `/metaDataList` | OPT_OR: roleValidator(DEFAULT) | 元数据列表（含预发布状态过滤），带 @OperLog(READ_ACCESS) |
| 13 | POST | `/userList` | OPT_OR: roleValidator(DEFAULT) | 用户可见模板列表（SYS_READER 过滤），带 @OperLog(READ_ACCESS) |
| 14 | POST | `/getDocTemplateListByUser` | OPT_OR: roleValidator(DEFAULT) | 查询用户相关有文档的模板，带 @OperLog(READ_ACCESS) |
| 15 | POST | `/checkAvailable` | OPT_OR: roleValidator(DEFAULT) | 检查模板是否可用 |
| 16 | POST | `/checkCodeUniqueness` | OPT_OR: roleValidator(DEFAULT) \| roleValidator(ADMIN) | 检查编码唯一性 |
| 17 | POST | `/getTemplateNames` | 无权限注解 | 批量获取模板名称（Map<id, name>） |
| 18 | POST | `/getMechAuthToken` | @CompsiteValidators(OPT_AND): {OPT_OR: roleValidator(SETTING) \| roleValidator(ADMIN)} AND {OPT_OR: roleValidator(ADMIN) \| roleValidator(CATEGORY_ADMIN) \| authFieldValidator(CREATOR, SYS_EDITOR) \| categoryMaintainerAuthValidator(categoryId, templateId)} | 获取机制编辑 token |
| 19 | POST | `/batchUpdatePerm` | OPT_OR: roleValidator(DEFAULT) \| roleValidator(ADMIN) | 批量修改模板权限（异步） |
| 20 | POST | `/getBatchUpdatePermProgress` | 无权限注解 | 获取批量修改权限进度 |
| 21 | POST | `/exportTemplate` | OPT_OR: roleValidator(CATEGORY_ADMIN, ADMIN) \| TemplateEditorValidator(categoryId, templateId) | 导出模板配置，带 @OperLog(READ_ACCESS) |
| 22 | POST | `/asyncImportTemplate` | OPT_OR: roleValidator(DEFAULT) \| roleValidator(ADMIN) | 异步导入模板 |
| 23 | POST | `/importInterrupt` | OPT_OR: roleValidator(DEFAULT) \| roleValidator(ADMIN) | 取消导入任务 |
| 24 | POST | `/process` | OPT_OR: roleValidator(DEFAULT) \| roleValidator(ADMIN) | 获取导入进度 |

**易踩坑**
- `prePublish` 的 OPT_AND 是两个条件都必须满足，与 `publish` 的 OPT_OR 不同，混淆后权限校验会失效
- `batchUpdatePerm` 是异步的，调用方需轮询 `/getBatchUpdatePermProgress`，不能假设立即生效
- `getMechAuthToken` 权限最复杂，外层 OPT_AND 内嵌两个 OPT_OR 分支，照抄时注意括号层级
