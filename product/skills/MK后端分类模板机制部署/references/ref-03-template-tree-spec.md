# 模板树查询能力实现规范（ref-03）

> 本文档覆盖模板树查询的完整实现：树节点 VO、SearchService、Controller 三个树端点。
> 基础模板实现（Entity / VO / API / Repository / Service / Controller CRUD）参见 **[ref-02-template-base-spec.md](ref-02-template-base-spec.md)**。
> 占位符含义参见主 SKILL 的**四、占位符说明**。

***

## 端点 → Service 方法对照

| Controller 端点         | Service 方法                   | 使用场景         |
| --------------------- | ---------------------------- | ------------ |
| `POST getMgmtTree`    | `SearchService.getTree`      | 后台管理分类树（懒加载） |
| `POST getRuntimeTree` | `SearchService.getTreeInRun` | 前台运行时选模板     |
| `POST treeSearch`     | `SearchService.treeSearch`   | 关键词搜索模板      |

***

## 步骤0：创建模板树过滤类型枚举

**核心要点**：
- 放在 `{模块Key}-api` 模块的 `constant` 包下，供 SearchService 的 `filterByTreeFilterType` 方法引用
- 实现 `IEnum<Integer>`，当前只有 `withoutUnpublishStatu` 一个值；扩展其他过滤策略时追加枚举值并在 `filterByTreeFilterType` 中补充 case
- 注意：枚举值名称拼写为 `withoutUnpublishStatu`（`Status` 少一个 `s`），保持与已有代码一致

```java
/** 模板树节点过滤类型枚举，供 SearchService.filterByTreeFilterType 使用 */
@AllArgsConstructor
public enum TemplateTreeFilterType implements IEnum<Integer> {

    /** 过滤掉未发布（非 PUBLISH 状态）的模板节点 */
    withoutUnpublishStatu(1, "过滤掉未发布的表单模板");

    /** 枚举整型 code，数据库存储用 */
    @Getter
    private Integer code;

    /** 枚举描述 */
    @Getter
    private String messageKey;

    @Override
    public Integer getValue() {
        return code;
    }

    public static class Converter extends IEnum.Converter<Integer, TemplateTreeFilterType> {
    }
}
```

***

## 步骤1：创建树节点相关 VO

**核心要点**：

- `{Module}TemplateTreeRequestVO` 继承 `TreeParamDTO`（已内置 `parentId`、`searchWord`、`filters`、`sorts`）
- `{Module}TemplateTreeRequestVO` 依赖步骤0的 `TemplateTreeFilterType` 枚举，**必须先完成步骤0**
- `{Module}TemplateTreeNodeVO` 继承 `CommonTreeNodeVO`（ref-01 步骤3b 新建），追加树专属字段
- 两个类放在 `{模块Key}-api` 模块的 `dto.tree` 包下

```java
/** 模板树查询请求 VO */
@Getter @Setter @ToString
public class {Module}TemplateTreeRequestVO extends TreeParamDTO {

    /** true=统计各分类下文档数量，回填 docNum，默认 false */
    @ApiModelProperty("是否统计各分类下文档数量")
    private Boolean isCount = false;

    /**
     * 模板过滤类型，供 SearchService.filterByTreeFilterType 使用。
     * 依赖步骤0的 TemplateTreeFilterType 枚举（同包 constant 目录下）。
     */
    @ApiModelProperty("过滤类型")
    private TemplateTreeFilterType fdTemplateTreeFilterType;

    /** 是否注入分类可见性权限过滤，默认 false */
    @ApiModelProperty("是否过滤分类权限")
    private Boolean isFilterAuth = false;

    /** 配合系统配置 isShowEmptyData 使用，true=过滤无文档的空分类，默认 false */
    @ApiModelProperty("是否过滤无文档的空分类")
    private Boolean filterEmptyDataCategory = false;

    /** true=发起场景，过滤禁止新建的模板 */
    @ApiModelProperty("是否为发起场景")
    private Boolean isCreate;

    @ApiModelProperty("文档统计类型")
    private DocStatisticsType fdStatisticsType;

    @ApiModelProperty("统计规则码")
    private String ruleCode;
}
```

```java
/** 模板树节点 VO；nodeType=CATEGORY 时 value 为分类 fdId，nodeType=TEMPLATE 时为模板 fdId */
@Getter @Setter @ToString
public class {Module}TemplateTreeNodeVO extends CommonTreeNodeVO {

    /** 搜索场景 ONLYROOTANDPARENT 模式时填充，供前端面包屑展示第一级 */
    @ApiModelProperty("根节点")
    private CommonTreeNodeVO root;

    @ApiModelProperty("分类下的文档数量")
    private Long docNum;

    /** 仅模板节点有效；是否已收藏为常用模板 */
    @ApiModelProperty("是否已收藏")
    private Boolean fdCommonly;
}
```

***

## 步骤2：创建 SearchService

**核心要点**：

- 独立的 `@Component`，**不继承框架基类**，标注 `@Transactional(readOnly = true)`
- 依赖：`I{Module}TemplateApi`、`{Module}CategoryService`、`I{Module}CategoryRepository`、`{Module}CommonlyTemplateService`、`{Module}ConfigService`、`{Module}DocCountService`
- `treeSearch` 被 `getTree` 内部复用，两种调用模式通过 `SearchReturnType` 区分父路径填充深度
- `findCategoryTree` 是 `public` 方法，供外部复用

```java
// 本机制新建类的自模块导入：
import {modulePackage}.dto.tree.{Module}TemplateTreeNodeVO;
import {modulePackage}.dto.tree.{Module}TemplateTreeRequestVO;
import {modulePackage}.dto.tree.CommonTreeNodeVO;
import {modulePackage}.dto.tree.SearchReturnType;
// 其他依赖包参见 ref-00-bundle-spec.md

@Slf4j
@Component
@Transactional(readOnly = true, rollbackFor = Exception.class)
public class {Module}SearchService {

    @Autowired
    private I{Module}TemplateApi templateApi;

    @Autowired
    private {Module}CategoryService categoryService;

    @Autowired
    private I{Module}CategoryRepository categoryRepository;

    /** 常用模板服务：treeSearch 中用于标记 fdCommonly */
    @Autowired
    private {Module}CommonlyTemplateService commonlyTemplateService;

    /** 模块配置服务：getTreeInRun 中读取 isDataDesignDisplay / isShowEmptyData 开关 */
    @Autowired
    private {Module}ConfigService configService;

    /** 文档计数服务：getTreeInRun 中统计分类下文档数或过滤空分类 */
    @Autowired
    private {Module}DocCountService docCountService;

    // ---- getTree：对应 Controller.getMgmtTree -------------------------

    /**
     * 管理后台分类+模板树（懒加载）。
     * 先查当前父节点下的分类；parentId 或 searchWord 不为空时，追加已发布模板节点。
     * isCreate=true 或 isFilterAuth=true 时注入分类可见性过滤。
     */
    public List<{Module}TemplateTreeNodeVO> getTree({Module}TemplateTreeRequestVO treeRequest) {
        List<{Module}TemplateTreeNodeVO> rs = new ArrayList<>();
        if (Boolean.TRUE.equals(treeRequest.getIsCreate())
                || Boolean.TRUE.equals(treeRequest.getIsFilterAuth())) {
            // 注入分类可见性过滤器，仅返回当前用户有权限的分类
            treeRequest.getFilters().put({Module}CategoryFilter.ID, null);
        }
        JSONArray categoryArray = findCategoryTree(treeRequest);
        if (CollUtil.isNotEmpty(categoryArray)) {
            for (int i = 0; i < categoryArray.size(); i++) {
                rs.add(createCategoryTreeNode(categoryArray.getJSONObject(i)));
            }
        }
        String parentId = treeRequest.getParentId();
        if (StrUtil.isNotBlank(parentId) || StrUtil.isNotBlank(treeRequest.getSearchWord())) {
            QueryRequest request = new QueryRequest();
            if (StrUtil.isNotBlank(parentId)) {
                request.addCondition("fdCategory.fdId", Operator.eq, parentId);
            } else {
                // searchWord 不为空时全库模糊搜索模板名称
                request.addCondition("fdName", Operator.contains, treeRequest.getSearchWord());
            }
            if (BooleanUtils.isTrue(treeRequest.getIsCreate())) {
                // 发起场景：过滤禁止新建的模板
                request.addCondition("fdSettingList.fdDisableCreate", Operator.neq, true);
            }
            request.addFilter(AuthConstant.AUTH_QUERYFILTER, AuthConstant.SYS_READER);
            // ONLYROOTANDPARENT：只填根节点和直接父节点，减少递归查询
            rs.addAll(treeSearch(request, SearchReturnType.ONLYROOTANDPARENT));
        }
        return rs;
    }

    // ---- getTreeInRun：对应 Controller.getRuntimeTree ----------------------

    /**
     * 运行时分类+模板树（用户发起主文档场景）。
     * 在 getTree 基础上追加：① 权限过滤 ② 文档计数 or 空分类过滤 ③ childFlag 回填。
     */
    public List<{Module}TemplateTreeNodeVO> getTreeInRun({Module}TemplateTreeRequestVO treeRequest) {
        if (treeRequest.getIsFilterAuth()) {
            treeRequest.getFilters().put({Module}CategoryFilter.ID, null);
        }
        List<{Module}TemplateTreeNodeVO> rs = getTree(treeRequest);
        if (CollUtil.isEmpty(rs)) {
            return rs;
        }
        // 文档计数 or 空分类过滤（二选一）
        // isCount 分支：需全局配置 isDataDesignDisplay=true 才生效
        if (StrUtil.isBlank(treeRequest.getSearchWord()) && treeRequest.getIsCount()
                && (configService.getConfig() != null
                        && Boolean.TRUE.equals(configService.getConfig().getIsDataDesignDisplay()))) {
            Map<String, Long> countMap = docCountService.countMainByCategory(
                    treeRequest.getParentId(), treeRequest.getFdStatisticsType(), treeRequest.getRuleCode());
            if (CollUtil.isNotEmpty(countMap)) {
                rs.forEach(node -> node.setDocNum(countMap.get(node.getValue())));
            }
        } else if (Boolean.TRUE.equals(treeRequest.getFilterEmptyDataCategory())
                // 空分类过滤分支：需全局配置 isShowEmptyData=false 才生效
                && (configService.getConfig() != null
                        && !Boolean.TRUE.equals(configService.getConfig().getIsShowEmptyData()))) {
            Set<String> idSet = docCountService.filterEmptyDataCategory(
                    treeRequest.getParentId(), treeRequest.getFdStatisticsType());
            // 移除不在有效集合中的节点，同时移除 value 为空的 CATEGORY 节点
            rs.removeIf(node -> !idSet.contains(node.getValue())
                    || (Objects.equals(node.getNodeType(), NodeType.CATEGORY)
                            && !StringUtils.hasText(node.getValue())));
        }
        if (CollUtil.isEmpty(rs)) {
            return rs;
        }
        // childFlag 回填：有子分类 OR 有已发布模板 → 前端显示展开箭头
        List<String> categoryIds = rs.stream()
                .filter(node -> NodeType.CATEGORY.equals(node.getNodeType()))
                .map(CommonTreeNodeVO::getValue)
                .collect(Collectors.toList());
        if (CollUtil.isEmpty(categoryIds)) {
            return rs;
        }
        Set<String> withChildSet = categoryRepository.filterWithChild(categoryIds);
        // 通过 API 接口调用，避免 SearchService 直接依赖 templateRepository
        Set<String> withTemplateSet = templateApi.filterCategoryIdWithTemplate(categoryIds);
        for ({Module}TemplateTreeNodeVO node : rs) {
            if (!NodeType.CATEGORY.equals(node.getNodeType())) {
                continue;
            }
            node.setChildFlag(withChildSet.contains(node.getValue())
                    || withTemplateSet.contains(node.getValue()));
        }
        return rs;
    }

    // ---- treeSearch：对应 Controller.treeSearch，也被 getTree 内部复用 ------

    /** Controller 入口，使用 ALLPARENT 填充完整父路径。 */
    public List<{Module}TemplateTreeNodeVO> treeSearch(QueryRequest queryRequest) {
        return treeSearch(queryRequest, SearchReturnType.ALLPARENT);
    }

    /**
     * 模板树查询核心实现。
     * ALLPARENT：递归填充完整祖先链（Controller 直接调用时使用）。
     * ONLYROOTANDPARENT：只填根节点和直接父节点（getTree 内部调用时使用，减少查询）。
     */
    public List<{Module}TemplateTreeNodeVO> treeSearch(QueryRequest queryRequest,
                                                       SearchReturnType searchReturnType) {
        List<{Module}TemplateTreeNodeVO> rs = new ArrayList<>();
        // 委托 findTemplateTree 统一处理分页、列投影、排序、权限过滤
        {Module}TemplateTreeRequestVO treeRequestVO = new {Module}TemplateTreeRequestVO();
        QueryResult<{Module}TemplateVO> result = findTemplateTree(queryRequest, treeRequestVO);
        // 查询当前用户的常用模板集合，用于标记 fdCommonly
        Set<String> commonlyIdSet = commonlyTemplateService.myList(null).stream()
                .map({Module}CommonlyTemplateVO::getFdTemplate)
                .map({Module}TemplateVO::getFdId)
                .collect(Collectors.toSet());
        for ({Module}TemplateVO templateVO : result.getContent()) {
            // 委托过滤方法：withoutUnpublishStatu 过滤未发布模板
            if (filterByTreeFilterType(templateVO, TemplateTreeFilterType.withoutUnpublishStatu)) {
                continue;
            }
            {Module}TemplateTreeNodeVO node = new {Module}TemplateTreeNodeVO();
            node.setValue(templateVO.getFdId());
            node.setText(templateVO.getFdName());
            node.setNodeType(NodeType.TEMPLATE);
            node.setFdIcon(templateVO.getFdIcon());
            // 模板自身的 fdHierarchyId = 所属分类 hierarchyId + 模板 fdId + "/"
            node.setFdHierarchyId(templateVO.getFdCategory().getFdHierarchyId()
                    + templateVO.getFdId() + TreeEntity.HIERARCHY_ID_SPLIT);
            // getFdCategory() 返回 {Module}CategoryVO，注意第二个参数类型
            fillTreeNodeParent(node, templateVO.getFdCategory(), searchReturnType);
            // 标记是否为当前用户的常用模板
            node.setFdCommonly(commonlyIdSet.contains(templateVO.getFdId()));
            rs.add(node);
        }
        return rs;
    }

    // ---- 私有/包级辅助方法 --------------------------------------------------

    /**
     * 模板过滤策略方法。
     * withoutUnpublishStatu：返回 true 表示该模板应被过滤（状态非 PUBLISH）。
     * 扩展其他过滤类型时在此追加 case。
     */
    private boolean filterByTreeFilterType({Module}TemplateVO templateVO,
                                           TemplateTreeFilterType filterType) {
        if (filterType == TemplateTreeFilterType.withoutUnpublishStatu) {
            // 过滤掉未发布的模板（DRAFT / PREPUBISH / DISABLE）
            return !TemplateStatus.PUBLISH.equals(templateVO.getFdStatus());
        }
        return false;
    }

    /**
     * 查询分类树并批量回填 childFlag（当前层分类是否有子分类）。
     * public：供外部场景复用（如分类选择器）。
     * 注意：getTreeInRun 会在此基础上再追加"有已发布模板"条件，最终 childFlag 以 getTreeInRun 为准。
     */
    public JSONArray findCategoryTree(TreeParamDTO treeParamDTO) {
        treeParamDTO.addColumn("fdIcon");
        if (CollUtil.isEmpty(treeParamDTO.getSorts())) {
            treeParamDTO.addSort("fdOrder", QueryConstant.Direction.ASC);
            treeParamDTO.addSort("fdId", QueryConstant.Direction.ASC);
        }
        JSONArray tree = categoryService.tree(treeParamDTO);
        if (CollUtil.isEmpty(tree)) {
            return tree;
        }
        List<String> parentIds = new ArrayList<>(tree.size());
        for (int i = 0; i < tree.size(); i++) {
            String id = tree.getJSONObject(i).getString("value");
            if (StrUtil.isNotBlank(id)) {
                parentIds.add(id);
            }
        }
        if (CollUtil.isNotEmpty(parentIds)) {
            Set<String> parentIdSet = categoryRepository.filterWithChild(parentIds);
            for (int i = 0; i < tree.size(); i++) {
                JSONObject category = tree.getJSONObject(i);
                category.put("childFlag", parentIdSet.contains(category.getString("value")));
            }
        }
        return tree;
    }

    /**
     * 查询已发布模板列表，统一处理列投影、排序、权限过滤。
     * 非管理员：SYS_EDITOR 过滤转换为 {Module}TemplateEditorFilter，其余默认加 SYS_READER。
     * 管理员/分类管理员：移除权限过滤，返回全量数据。
     * isFilterAuth=true 时，非管理员额外追加 SYS_READER 过滤（独立判断，不受 isAdmin 分支影响）。
     */
    private QueryResult<{Module}TemplateVO> findTemplateTree(QueryRequest queryRequest,
                                                              {Module}TemplateTreeRequestVO treeRequest) {
        queryRequest.setPageSize(QueryRequest.MAX_PAGESIZE);
        queryRequest.addColumn("fdId", "fdName", "fdCategory.fdId", "fdCategory.fdHierarchyId",
                "fdCategory.fdTreeLevel", "fdCategory.fdName", "fdDesc", "fdIcon", "fdStatus");
        queryRequest.addSort("fdOrder", QueryConstant.Direction.ASC);
        queryRequest.addSort("fdId", QueryConstant.Direction.DESC);

        // 按 parentId 限定分类范围（treeSearch Controller 入口不传 parentId，此处为空则跳过）
        String parentId = treeRequest.getParentId();
        if (!StringUtils.isEmpty(parentId)) {
            queryRequest.addCondition("fdCategory.fdId", Operator.eq, parentId);
        }
        // 只查已发布模板；不查禁止新建的模板
        queryRequest.addCondition("fdStatus", Operator.eq, 1);
        queryRequest.addCondition("fdSettingList.fdDisableCreate", Operator.neq, true);

        LinkedHashMap<String, Object> filters = queryRequest.getFilters();
        Object authFilter = (CollUtil.isNotEmpty(filters) && Objects.nonNull(filters.get(AuthConstant.AUTH_QUERYFILTER)))
                ? filters.get(AuthConstant.AUTH_QUERYFILTER) : null;

        boolean isAdmin = UserUtil.checkRole({Module}Role.ROLE_{MODULE}_ADMIN)
                || UserUtil.checkRole({Module}Role.ROLE_{MODULE}_CATEGORY_ADMIN);
        if (!isAdmin) {
            // 非管理员：将 SYS_EDITOR 转换为编辑者过滤器，其余保持 SYS_READER
            if (Objects.nonNull(authFilter) && authFilter instanceof String) {
                filters.remove(AuthConstant.AUTH_QUERYFILTER);
                if (AuthConstant.SYS_EDITOR.equals(authFilter)) {
                    queryRequest.addFilter({Module}TemplateEditorFilter.ID, null);
                } else {
                    queryRequest.addFilter(AuthConstant.AUTH_QUERYFILTER, AuthConstant.SYS_READER);
                }
            }
        } else {
            // 管理员：移除权限过滤，返回全量
            if (Objects.nonNull(authFilter)) {
                filters.remove(AuthConstant.AUTH_QUERYFILTER);
            }
        }
        // isFilterAuth=true 时，非管理员额外追加 SYS_READER 过滤（独立于上方 isAdmin 分支）
        if (treeRequest.getIsFilterAuth() != null && treeRequest.getIsFilterAuth()) {
            if (!isAdmin) {
                queryRequest.addFilter(AuthConstant.AUTH_QUERYFILTER, AuthConstant.SYS_READER);
            }
        }
        return templateApi.findAll(queryRequest);
    }

    /** 将分类 JSON 节点转换为树节点 VO（nodeType=CATEGORY）。 */
    private {Module}TemplateTreeNodeVO createCategoryTreeNode(JSONObject obj) {
        {Module}TemplateTreeNodeVO node = new {Module}TemplateTreeNodeVO();
        node.setValue(obj.getString("value"));
        node.setText(obj.getString("text"));
        node.setDesc(obj.getString("desc"));
        node.setNodeType(NodeType.CATEGORY);
        node.setFdHierarchyId(obj.getString("fdHierarchyId"));
        node.setFdIcon(obj.getString("fdIcon"));
        node.setFdOrder(obj.getInteger("fdOrder"));
        node.setParent(obj.getObject("parent", CommonTreeNodeVO.class));
        // root 由框架分类树接口填充（当前节点的根分类引用）
        node.setRoot(obj.getObject("root", CommonTreeNodeVO.class));
        return node;
    }

    /**
     * 填充模板节点的父分类引用。
     * 注意：第二个参数类型是 {Module}**Category**VO（分类 VO），不是 {Module}TemplateVO。
     * ALLPARENT：递归填充完整祖先链（Controller 直接调用时使用）。
     * ONLYROOTANDPARENT：只填根节点（root）和直接父节点（parent），减少查询。
     *
     * @param node             待填充的模板树节点
     * @param fdCategory       模板所属分类 VO（来自 templateVO.getFdCategory()，类型为 {Module}CategoryVO）
     * @param searchReturnType 父路径填充深度
     */
    private void fillTreeNodeParent({Module}TemplateTreeNodeVO node,
                                    {Module}CategoryVO fdCategory,   // 注意：CategoryVO，非 TemplateVO
                                    SearchReturnType searchReturnType) {
        Integer treeLevel = fdCategory.getFdTreeLevel();
        String hierarchyId = fdCategory.getFdHierarchyId();
        switch (searchReturnType) {
            case ALLPARENT:
                categoryService.fillTreeNodeParent(node, hierarchyId);
                break;
            case ONLYROOTANDPARENT:
                String[] ids = hierarchyId.split(TreeEntity.HIERARCHY_ID_SPLIT);
                List<String> idList = Arrays.stream(ids)
                        .filter(StrUtil::isNotBlank)
                        .collect(Collectors.toList());
                node.setRoot(categoryService.getTreeNodeVOByCategoryId(idList.get(0)));
                if (treeLevel != null && treeLevel > 1) {
                    node.setParent(categoryService.buildTreeNodeWithCategoryVO(fdCategory));
                }
                break;
            default:
                break;
        }
    }
}
```

***

## 步骤3：在 Controller 中声明三个树端点

**核心要点**：

- 在已有 `{Module}TemplateController` 中注入 `{Module}SearchService` 并追加三个端点
- 权限注解按业务实际添加，此处仅展示方法骨架

```java
// 本机制新建类的自模块导入：
import {modulePackage}.dto.tree.{Module}TemplateTreeNodeVO;
import {modulePackage}.dto.tree.{Module}TemplateTreeRequestVO;
// 其他依赖包参见 ref-00-bundle-spec.md

// 注入 SearchService（在已有 Controller 中追加）
@Autowired
private {Module}SearchService searchService;

/** 后台管理分类+模板树（懒加载），需管理权限。 */
@PostMapping("getMgmtTree")
public Response<List<{Module}TemplateTreeNodeVO>> getMgmtTree(
        @RequestBody {Module}TemplateTreeRequestVO treeRequest) {
    return Response.ok(searchService.getTree(treeRequest));
}

/** 运行时分类+模板树（用户发起场景），含权限过滤、文档计数/空分类过滤、childFlag 回填。 */
@PostMapping("getRuntimeTree")
public Response<List<{Module}TemplateTreeNodeVO>> getRuntimeTree(
        @RequestBody {Module}TemplateTreeRequestVO treeRequest) {
    return Response.ok(searchService.getTreeInRun(treeRequest));
}

/** 关键词搜索已发布模板，结果节点携带 parent 父分类引用。 */
@PostMapping("treeSearch")
public Response<List<{Module}TemplateTreeNodeVO>> treeSearch(
        @RequestBody QueryRequest queryRequest) {
    return Response.ok(searchService.treeSearch(queryRequest));
}
```

***

## 调用链路

```
Controller.getMgmtTree
  └─ SearchService.getTree
       ├─ findCategoryTree()                     → 分类节点（含 childFlag）
       └─ treeSearch(request, ONLYROOTANDPARENT) → 模板节点（parentId 或 searchWord 不为空时）

Controller.getRuntimeTree
  └─ SearchService.getTreeInRun
       ├─ SearchService.getTree(...)             → 复用上述逻辑
       ├─ docCountService.countMainByCategory()  → 回填 docNum（isCount=true 时）
       ├─ docCountService.filterEmptyDataCategory() → 过滤空分类（filterEmptyDataCategory=true 时）
       └─ filterWithChild() + filterCategoryIdWithTemplate() → 设置 childFlag

Controller.treeSearch
  └─ SearchService.treeSearch(request, ALLPARENT)
       └─ templateApi.findAll()                  → 仅模板节点（完整父路径）
```

***

## 注意事项

1. **`findCategoryTree`** **的** **`childFlag`** **只考虑子分类**：`getTreeInRun` 会追加"有已发布模板"条件，最终值以 `getTreeInRun` 的设置为准
2. **`treeSearch`** **内部硬编码只查已发布模板**：调用方无需额外传入 `fdStatus` 条件
3. **`filterCategoryIdWithTemplate`** **通过 API 调用**：`SearchService` 不直接依赖 `templateRepository`，保持层次隔离
4. **`ONLYROOTANDPARENT`** **性能优化**：`getTree` 内部复用 `treeSearch` 时使用此模式，避免递归查询完整祖先链
5. **文档计数服务按业务引入**：`isCount`/`filterEmptyDataCategory` 逻辑需注入业务文档计数服务，代码中已给出示例注释

