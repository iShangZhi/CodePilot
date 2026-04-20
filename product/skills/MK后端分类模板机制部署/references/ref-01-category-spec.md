# 分类机制部署步骤（ref-01）

> 占位符含义参见主 SKILL 的**四、占位符说明**。
> 所有跨模块包导入以 [ref-00-bundle-spec.md](ref-00-bundle-spec.md) 为准；各步骤代码块中仅保留本机制新建类的导入。

***

## 步骤1：创建分类实体 Entity

**核心要点**：

- 实现 `CategoryEntity<自身类型, 权限实体类型>` 接口（框架接口，继承 `BaseCategoryEntity` + `PermissionEntity`），框架自动提供 `fdName`、`fdDesc`、`fdOrder`、`fdParent`、`fdHierarchyId`、`fdTreeLevel`、`fdPermissions`、`fdReaderFlag` 等核心字段，无需手动声明
- `@MetaEntity` 用于元数据配置，`displayProperty` 指定列表/树中显示的字段名，`messageKey` 指定国际化 key
- 扩展字段使用 `@MetaProperty` 注册元数据，`langSupport` 默认不开启多语言
- `CategoryEntity`（通过 `TreeEntity`）已自动管理 `fdHierarchyId`，无需在 `beforeSaveOrUpdate` 中手动维护层级路径
- **可选接口 `FdIsTransferData`**：若模块需要支持数据迁移（如 EKP 迁移），Entity 应额外实现此接口并添加 `@SysTransferEntity` 注解
- **可选注解 `@AuthFieldFilters`**：用于字段级权限过滤（如按创建者过滤），通过 `@AuthFieldFilter(fieldName = "fdCreator.fdId", filterType = "creator")` 声明
- **`fdDesc` 若需支持富文本/长文本**：需在 Entity 中显式重新声明 `fdDesc` 字段并加 `@Lob` 注解（框架默认 `fdDesc` 为普通字符串）

| 继承/接口 | 说明 |
|-----------|------|
| `CategoryEntity<自身类型, 权限实体类型>` | 框架接口，继承 `BaseCategoryEntity`（提供 `fdName`、`fdDesc`、`fdOrder`、`fdParent`、`fdHierarchyId`、`fdTreeLevel`）+ `PermissionEntity`（提供 `fdPermissions`、`fdReaderFlag`） |
| `FdIsTransferData`（可选） | 数据迁移标记接口，配合 `@SysTransferEntity` 注解使用 |

```java
// 本机制新建类无需额外导入；所有依赖包参见 ref-00-bundle-spec.md

@Getter
@Setter
@Entity
@Table
@MetaEntity(
    displayProperty = "fdName",
    messageKey = "{modulePrefix}:table.{module}Category"
)
// 可选：字段级权限过滤，按创建者过滤
// @AuthFieldFilters(filters = {
//     @AuthFieldFilter(fieldName = "fdCreator.fdId", filterType = "creator"),
// })
// 可选：数据迁移支持
// @SysTransferEntity
public class {Module}Category extends AbstractEntity
        implements CategoryEntity<{Module}Category, {Module}CategoryPerm>
        /* 可选：, FdIsTransferData */ {

    @MetaProperty(
        messageKey = "{modulePrefix}:{module}Category.fdIcon"
        // langSupport 默认 false；如需多语言设为 true
    )
    private String fdIcon;

    // 可选：若 fdDesc 需要支持富文本/长文本，显式重新声明并加 @Lob
    // @Lob
    // @MetaProperty(messageKey = "{modulePrefix}:meta.desc", langSupport = true)
    // private String fdDesc;

    /** 可阅读者、可编辑者权限集合 */
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true, mappedBy = "fdMainDoc")
    @JsonIgnore
    @JSONField(serialize = false)
    private List<{Module}CategoryPerm> fdPermissions;
}
```

***

## 步骤2：创建分类权限实体 Entity

**核心要点**：

- 继承 `PermissionInfoEntity<分类实体>` 即可，框架自动处理权限关联
- **只需创建 Entity，无需 VO、API、Service、Controller**
- 用于表级（行级）权限控制

```java
// 本机制新建类无需额外导入；所有依赖包参见 ref-00-bundle-spec.md

@Getter
@Setter
@Entity
@Table
public class {Module}CategoryPerm extends PermissionInfoEntity<{Module}Category> {
}
```

> **注意**：此实体仅供框架内部权限控制使用，不对外暴露 API。

***

## 步骤3：创建分类 VO

**核心要点**：

- 继承 `AbstractCategoryVO`，已自动包含 `fdName`、`fdDesc`、`fdOrder`、`fdParent`、`fdHierarchyId`、`fdTreeLevel`、`showType` 等核心字段，无需重复声明
- 只添加实体中新增的业务扩展字段

```java
// 本机制新建类无需额外导入；所有依赖包参见 ref-00-bundle-spec.md

@Getter
@Setter
@ToString(callSuper = true)
@ApiModel("{module}分类VO")
public class {Module}CategoryVO extends AbstractCategoryVO {

    @ApiModelProperty(value = "分类图标")
    private String fdIcon;
}
```

***

## 步骤3b：创建分类树节点 VO

**核心要点**：
- `CommonTreeNodeVO` 是**业务模块自定义的通用树节点基类**，需在本模块 `dto.tree` 包下新建，不是框架类；内置 `text`、`value`、`fdHierarchyId`、`nodeType`、`canRead`、`parent` 等字段，`parent` 类型为自身，形成链表结构
- `{Module}TreeNodeVO extends CommonTreeNodeVO`：`buildTreeNodeWithCategoryVO` 内部 `new` 的是子类实例，用于区分模块节点类型，同时便于追加业务扩展字段（如收藏数、文档数等）；API/Service/Controller 的签名统一使用父类 `CommonTreeNodeVO`
- 两个类均放在 `{模块Key}-api` 模块的 `dto.tree` 包下，供 api 和 core 共同引用

**`CommonTreeNodeVO`**（放在 `{modulePackage}.dto.tree`）：

```java
package {modulePackage}.dto.tree;

// 依赖包参见 ref-00-bundle-spec.md（Hutool / Landray Sys Category / Lombok）

/**
 * 通用树节点，用于 loadParentNode 返回祖先链（parent 引用逐级向上形成链表，供面包屑/路径回显使用）。
 */
@Getter
@Setter
@ToString
public class CommonTreeNodeVO implements Comparable<CommonTreeNodeVO> {

    private String text;
    private String desc;
    private String value;
    private String fdIcon;
    /** 格式 "/id1/id2/id3/"，用于前缀匹配所有后代节点 */
    private String fdHierarchyId;
    private NodeType nodeType;
    private Integer fdOrder;
    /** true 时前端显示展开箭头 */
    private boolean childFlag;
    /** 当前用户是否可见该分类（由 checkCategoryReader 写入） */
    private Boolean canRead;
    /** 逐级向上填充形成链表，根节点为 null */
    private CommonTreeNodeVO parent;

    @Override
    public int compareTo(CommonTreeNodeVO o) {
        if (o == null) return -1;
        if (this.fdOrder == null && o.fdOrder == null) return 0;
        if (this.fdOrder != null && o.fdOrder == null) return -1;
        if (this.fdOrder == null) return 1;
        return CompareUtil.compare(this.fdOrder, o.fdOrder);
    }
}
```

**`{Module}TreeNodeVO`**（放在 `{modulePackage}.dto.tree`）：

```java
package {modulePackage}.dto.tree;

// 依赖包参见 ref-00-bundle-spec.md（Lombok）

/** {module} 分类树节点，继承 CommonTreeNodeVO，可按需追加业务扩展字段 */
@Getter
@Setter
@ToString
public class {Module}TreeNodeVO extends CommonTreeNodeVO {

    // 可按需追加业务扩展字段，例如：
    // private Long docNum;        // 文档数
    // private Boolean fdCommonly; // 是否收藏
}
```

***

## 步骤4：创建分类 API 接口

**核心要点**：

- 继承 `ICategoryApi<分类VO>`，自动获得标准的分类树查询、CRUD 等接口声明
- 框架基础部署只需继承 `ICategoryApi`，业务自定义方法（如拷贝分类、查父节点链等）由服务清单 SKILL 补充

```java
// 本机制新建类无需额外导入；所有依赖包参见 ref-00-bundle-spec.md

@Api(tags = "{module}分类服务")
public interface I{Module}CategoryApi extends ICategoryApi<{Module}CategoryVO> {

    // 业务自定义方法由服务清单 SKILL 补充
}
```

***

## 步骤5：创建分类 Repository

**核心要点**：

- 继承 `IRepository<分类实体>`，框架提供基础 CRUD
- 拷贝功能依赖以下三个自定义查询，必须实现：
  - `countSubCategoryByCategoryId`：统计直接子分类数量，用于 `delete` 前置校验
  - `findChildren`：查询直接子分类列表，用于 `copy` 递归复制
  - `countByFdName`：统计同名分类数量，用于拷贝时自动生成唯一名称（`BeanHelper.setAvailableValue`）

```java
@Repository
public interface I{Module}CategoryRepository extends IRepository<{Module}Category> {

    /**
     * 统计直接子分类数量（用于 delete 前置校验：大于 0 禁止删除）
     */
    @Query("select count(1) from {Module}Category where fdParent.fdId = ?1")
    long countSubCategoryByCategoryId(String categoryId);

    /**
     * 查询直接子分类列表（用于 copy 递归复制子树）
     * 使用原生 SQL（nativeQuery = true），若表名与约定不符需自行调整。
     */
    @Query(value = "select * from {module}_category where fd_parent_id = ?1", nativeQuery = true)
    List<{Module}Category> findChildren(String fdParentId);

    /**
     * 统计同名分类数量（配合 BeanHelper.setAvailableValue 保证拷贝名称唯一）
     * 分类无软删除，直接按 fdName 计数。
     */
    @Query("select count(1) from {Module}Category where fdName = ?1")
    long countByFdName(String fdName);

    /**
     * 从候选列表中过滤出拥有直接子分类的分类 ID 集合。
     * 用于 getTreeInRun 回填 childFlag（判断"有子分类"部分）。
     */
    @Query("select distinct c.fdParent.fdId from {Module}Category c where c.fdParent.fdId in ?1")
    Set<String> filterWithChild(List<String> categoryIds);
}
```

***

## 步骤6：创建分类 Service

**核心要点**：

- 同时实现 `ICategoryService` 和业务 API 接口，框架通过 `ICategoryService` 注入分类核心能力
- `@RestController` + `@RequestMapping` 使 Service 直接作为 Feign 服务端暴露
- 请求路径格式：`/api/{模块前缀}/{module}Category`
- **`getEntityManager` 必须重写**：返回 `entityManager`，框架分类树操作依赖此方法
- **`beforeSaveOrUpdate` 必须重写**：强制设置 `entity.setFdReaderFlag(false)`，关闭分类级阅读者标记（分类权限由 `fdPermissions` 独立控制，不走 `fdReaderFlag` 逻辑）
- **`entityToVo` / `voToEntity` 必须重写**：手动映射 `fdOrder` 字段，框架的 `super.entityToVo` / `super.voToEntity` **不会自动映射** `fdOrder`
- **`delete` 需重写**：调用 `super.delete` 前校验分类下是否有模板或子分类，有则抛出 `KmssRuntimeException`（框架校验模式）
- **注入 `{Module}TemplateService`**：`delete` 校验依赖模板 Service 的 `countByCategory` 方法

```java
// 本机制新建类无需额外导入；所有依赖包参见 ref-00-bundle-spec.md

@Slf4j
@Service
@RestController
@RequestMapping("/api/{modulePrefix}/{module}Category")
@Transactional(rollbackFor = Exception.class)
public class {Module}CategoryService
        extends AbstractServiceImpl<I{Module}CategoryRepository, {Module}Category, {Module}CategoryVO>
        implements ICategoryService<{Module}Category, {Module}CategoryVO>, I{Module}CategoryApi {

    @Autowired
    private {Module}TemplateService templateService;

    @Override
    public EntityManager getEntityManager() {
        return entityManager;
    }

    /**
     * 保存/更新前置处理：强制关闭 fdReaderFlag。
     * 分类权限由 fdPermissions（@OneToMany）独立控制，不走框架的 fdReaderFlag 逻辑。
     */
    @Override
    protected void beforeSaveOrUpdate({Module}Category entity, boolean isAdd) {
        entity.setFdReaderFlag(false);
        super.beforeSaveOrUpdate(entity, isAdd);
    }

    /**
     * Entity → VO 转换：手动映射 fdOrder。
     * 框架 super.entityToVo 不会自动映射 fdOrder，必须显式补充。
     */
    @Override
    protected void entityToVo({Module}Category entity, {Module}CategoryVO vo, boolean isAdd) {
        super.entityToVo(entity, vo, isAdd);
        vo.setFdOrder(entity.getFdOrder());
    }

    /**
     * VO → Entity 转换：手动映射 fdOrder。
     * 框架 super.voToEntity 不会自动映射 fdOrder，必须显式补充。
     */
    @Override
    protected void voToEntity({Module}CategoryVO vo, {Module}Category entity, boolean isAdd) {
        super.voToEntity(vo, entity, isAdd);
        entity.setFdOrder(vo.getFdOrder());
    }

    /** 删除前校验：有模板或子分类时禁止删除 */
    @Override
    public void delete(IdVO id) {
        if (checkHasTemplate(id)) {
            throw new KmssRuntimeException("error.dataExist", "该分类下有模板，请先删除模板才可以删除当前分类");
        }
        if (checkHasSubCategory(id)) {
            throw new KmssRuntimeException("error.dataExist", "该分类下有子分类，请先删除子分类才可以删除当前分类");
        }
        super.delete(id);
    }

    private boolean checkHasTemplate(IdVO id) {
        return templateService.countByCategory(id.getFdId()) > 0;
    }

    private boolean checkHasSubCategory(IdVO id) {
        return repository.countSubCategoryByCategoryId(id.getFdId()) > 0;
    }

    // 业务自定义方法（copy、loadParentNode、fillTreeNodeParent 等）由服务清单 SKILL 补充
}
```

***

## 步骤7：创建分类 Controller

**核心要点**：
- `CategoryController` = `CrudController` + `DeleteAllController` + `tree` 默认方法，**不含** `ListController` 和 `MetaController`
- 请求路径格式：`/data/{modulePrefix}/{module}Category`（前端直接调用此路径）
- `CategoryController#tree` 已有 `@PostMapping("tree")` 默认实现，**仍需在 Controller 中显式 `@Override` 以确保路由注册到当前 Controller**（Spring MVC 在多接口默认方法冲突时可能丢失路由）
- `ICategoryApi#search` **没有 `@PostMapping` 注解**，必须在 Controller 中显式声明才能暴露为 HTTP 端点

**需要显式声明的端点**：

| HTTP 端点 | 来源 | 说明 |
|-----------|------|------|
| `POST tree` | `CategoryController` default | 有默认实现，但需显式 `@Override` 确保路由生效 |
| `POST search` | `ICategoryApi#search` | **无 `@PostMapping`**，必须显式声明才能暴露为 HTTP 端点 |

```java
// 本机制新建类无需额外导入；所有依赖包参见 ref-00-bundle-spec.md

@RestController
@RequestMapping("/data/{modulePrefix}/{module}Category")
public class {Module}CategoryController
        extends AbstractController<I{Module}CategoryApi, {Module}CategoryVO>
        implements CategoryController<I{Module}CategoryApi, {Module}CategoryVO> {

    /**
     * 分类树（POST tree）
     * CategoryController 有 default 实现，但需显式 @Override 确保路由注册到当前 Controller。
     */
    @Override
    @PostMapping("tree")
    public Response<JSONArray> tree(@RequestBody TreeParamDTO vo) {
        return Response.ok(getApi().tree(vo));
    }

    /**
     * 关键词搜索分类树（POST search）
     * ICategoryApi#search 无 @PostMapping，必须在此显式声明才能暴露为 HTTP 端点。
     */
    @PostMapping("search")
    public Response<List<TreeNodeVO>> search(@RequestBody TreeRequestVO vo) {
        return Response.ok(getApi().search(vo));
    }

    // 业务自定义端点（copy、loadParentNode 等）由服务清单 SKILL 补充
}
```

***

## 步骤8：配置国际化资源

```properties
# 分类实体表名国际化
table.{module}Category=分类

# 分类字段国际化
{module}Category.fdName=分类名称
{module}Category.fdIcon=分类图标
{module}Category.fdDesc=分类描述
```

***

## 注意事项

1. **权限实体**：`{Module}CategoryPerm` 只需 Entity，无需 VO/API/Service/Controller
2. **国际化 key 格式**：`{modulePrefix}:{entityName}.{fieldName}`，前缀与模块包名前缀一致
3. **`delete` 依赖模板 Service 的方法**：`countByCategory(String categoryId)` 统计分类下的模板数量（用于 `delete` 前置校验），参见 ref-02 步骤5
4. **`findChildren` 使用原生 SQL**：实际实现使用 `nativeQuery = true` 查询 `fd_parent_id`，而非 JPQL 中的 `fdParent.fdId`。表名按框架约定为模块对应的下划线表名（如 `km_review_category`）
5. **`beforeSaveOrUpdate` 强制 `fdReaderFlag = false`**：分类权限完全由 `fdPermissions`（`@OneToMany` 关联的权限实体）控制，`fdReaderFlag` 必须关闭，否则框架可能启用基于阅读者的额外过滤逻辑，导致分类查询结果不符合预期
6. **`entityToVo` / `voToEntity` 手动映射 `fdOrder`**：框架的 `super.entityToVo` 和 `super.voToEntity` **不会自动映射** `fdOrder`（尽管 `CategoryEntity` 和 `AbstractCategoryVO` 都声明了该字段），必须在子类中显式补充 `vo.setFdOrder(entity.getFdOrder())` 和 `entity.setFdOrder(vo.getFdOrder())`；遗漏会导致分类排序信息丢失
7. **`FdIsTransferData` 接口**：若模块涉及数据迁移场景（如从旧系统导入分类数据），Entity 应实现 `FdIsTransferData` 并加 `@SysTransferEntity` 注解；非迁移场景可忽略
8. **`@AuthFieldFilters` 注解**：用于声明字段级权限过滤规则，常见配置为 `@AuthFieldFilter(fieldName = "fdCreator.fdId", filterType = "creator")`，表示按创建者进行权限过滤；该注解为可选，根据业务需要决定是否添加
9. **`{Module}TreeNodeVO` 继承 `CommonTreeNodeVO`**：API/Service/Controller 中的签名统一使用父类 `CommonTreeNodeVO`；内部实例化子类 `{Module}TreeNodeVO`，既可区分模块节点类型，又可按需追加业务扩展字段

