# ref-04 — CategoryUtil 工具类

> `{Module}CategoryUtil` 位于 `core.util` 包，静态工具类，通过 `ApplicationContextHolder` 懒加载 `I{Module}CategoryApi` Bean。

**核心规则**
- **只做条件转换，不做查询**：swapHierarchyId 本身不查数据，只改写 QueryRequest 中的条件操作符
- **入参为 child 时才生效**：非 child 操作符直接跳过，不抛异常
- **分类不存在时静默跳过**：loadById 返回 empty 时不转换，原条件保持不变，调用方需自行判断结果

---

## swapHierarchyId — child 条件转换为 startsWith

**使用场景**：前端传入 `{fdCategoryHierarchyId: {child: "分类ID"}}` 时，转换为按层级 ID 前缀匹配，实现"查分类下所有内容"语义。

```java
public class {Module}CategoryUtil {

    private static I{Module}CategoryApi {module}CategoryApi;

    private static I{Module}CategoryApi get{Module}CategoryApi() {
        if ({module}CategoryApi == null) {
            {module}CategoryApi = ApplicationContextHolder.getApplicationContext()
                .getBean(I{Module}CategoryApi.class);
        }
        return {module}CategoryApi;
    }

    /**
     * 将 child 查询条件转换为 startsWith，实现分类下所有内容查询。
     * 原理：用分类的 fdHierarchyId 前缀匹配替代 child 语义，避免递归查子分类 ID。
     *
     * @param request                    查询请求
     * @param fdCategoryHierarchyIdKey   entity 中层级 ID 的字段属性名
     */
    public static void swapHierarchyId(QueryRequest request, String fdCategoryHierarchyIdKey) {
        if (!CollectionUtils.isEmpty(request.getConditions())
                && request.getConditions().containsKey(fdCategoryHierarchyIdKey)) {
            Object value = request.getConditions().get(fdCategoryHierarchyIdKey);
            if (value instanceof Map) {
                Map<String, Object> hierarchyMap = (Map<String, Object>) value;
                if (hierarchyMap.containsKey(QueryConstant.Operator.child.getValue())) {
                    String parentId = (String) hierarchyMap
                        .get(QueryConstant.Operator.child.getValue());
                    Optional<{Module}CategoryVO> opt = get{Module}CategoryApi()
                        .loadById(IdVO.of(parentId));
                    if (opt.isPresent()) {
                        request.getConditions().remove(fdCategoryHierarchyIdKey);
                        request.addCondition(fdCategoryHierarchyIdKey, Operator.startsWith,
                                opt.get().getFdHierarchyId());
                    }
                }
            }
        }
    }
}
```

**易踩坑**
- `fdCategoryHierarchyIdKey` 是 entity 字段的**属性名**，不是数据库列名，传错会导致条件匹配不到、静默跳过
- 分类被删除后 loadById 返回 empty，此时原 child 条件未被替换，查询结果会包含所有分类数据（无过滤）
