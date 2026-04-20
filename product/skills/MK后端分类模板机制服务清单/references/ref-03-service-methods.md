# ref-03 — Service public 方法

> 不在 API 接口中定义、但为 public 的内部方法，供其他 Service 或 Controller 调用。

**核心规则**
- **copyByCategory 与 copyByTemplateSingle 是两个粒度**：前者复制分类下所有模板，后者复制单个模板（含机制），copy 是核心实现
- **isNotTempValid 带缓存**：清除缓存需配套调用 clearTempCache，否则状态不一致
- **SearchService 方法与 Controller 端点一一对应**：Controller 直接委托 SearchService，不在 Service 层做额外处理

---

## CategoryService 内部 public 方法

| # | 方法签名 | 说明 |
|---|---------|------|
| 1 | `void fillTreeNodeParent(CommonTreeNodeVO, String hierarchyId)` | 补全所有父节点 |
| 2 | `CommonTreeNodeVO buildTreeNodeWithCategoryVO({Module}CategoryVO)` | 根据分类构建树节点 |
| 3 | `CommonTreeNodeVO getTreeNodeVOByCategoryId(String categoryId)` | 根据 ID 获取树节点 |
| 4 | `String getCategoryPath(String categoryId)` | 获取分类路径字符串（"一级/二级/三级"） |

---

## TemplateService 内部 public 方法

| # | 方法签名 | 说明 |
|---|---------|------|
| 1 | `long countByCategory(String fdCategoryId)` | 统计分类下模板数量 |
| 2 | `void copyByCategory(Category source, Category target)` | 复制分类下所有模板 |
| 3 | `TemplateVO copyByTemplateSingle(TemplateVO resource)` | 复制单个模板（含机制） |
| 4 | `TemplateVO copy(TemplateVO source, CopyTemplateType type)` | 核心复制方法（含表单、编号、流程机制） |
| 5 | `boolean isNotTempValid(String templateId)` | 判断模板是否已发布（带缓存） |
| 6 | `void clearTempCache(String templateId)` | 清除模板有效性缓存 |
| 7 | `TemplateVO findByFdCode(String fdCode)` | 按编码查模板 |
| 8 | `Optional<TemplateVO> loadByFdCode(String fdCode)` | 按编码加载模板（含机制） |
| 9 | `void updateFdDeleted(IdVO id, Boolean fdDeleted)` | 更新软删标记 |
| 10 | `void updateFdState(String fdId, Boolean enable)` | 更新模板状态（启用/禁用） |
| 11 | `TemplateVO importTemplate(TemplateImportTask task)` | 导入模板（跨环境） |

---

## SearchService 内部 public 方法

| # | 方法签名 | 说明 |
|---|---------|------|
| 1 | `List<?> findCategoryTree(TreeParamDTO)` | 带过滤器的分类树查询 |
| 2 | `List<TemplateTreeNodeVO> getTree(TemplateTreeRequestVO)` | 管理后台分类+模板树 |
| 3 | `List<TemplateTreeNodeVO> treeSearch(QueryRequest)` | 关键词搜索模板树 |
| 4 | `List<TemplateTreeNodeVO> getTreeInRun(TemplateTreeRequestVO)` | 运行时分类+模板树 |
