# 模板基础能力部署步骤（ref-02）

> 占位符含义参见主 SKILL 的**四、占位符说明**。
> 所有跨模块包导入以 [ref-00-bundle-spec.md](ref-00-bundle-spec.md) 为准；各步骤代码块中仅保留本机制新建类的导入。

---

## 步骤0：创建模板状态枚举

**核心要点**：
- 放在 `{模块Key}-api` 模块的 `constant` 包下，供 Entity、VO、Service 统一引用
- 实现 `IEnum<Integer>`，框架通过内部 `Converter` 将枚举与数据库整型互转
- 状态流转：`DRAFT`（新建默认）→ `PREPUBISH`（预发布，可选）→ `PUBLISH`（已发布）→ `DISABLE`（已禁用）
- 注意：参考实现中拼写为 `PREPUBISH`（无第二个 `L`），保持与框架一致

```java
// 依赖包参见 ref-00-bundle-spec.md（Landray Common / Lombok）

/** 模板状态枚举：DRAFT → PREPUBISH → PUBLISH → DISABLE */
@AllArgsConstructor
public enum TemplateStatus implements IEnum<Integer> {

    /** 草稿（新建默认状态，未发布不可被主文档引用） */
    DRAFT(0, "{modulePrefix}:enum.TemplateStatus.DRAFT"),
    /** 已发布（可被主文档引用） */
    PUBLISH(1, "{modulePrefix}:enum.TemplateStatus.PUBLISH"),
    /** 已禁用（发布后禁用，现有主文档保留引用但禁止新建） */
    DISABLE(2, "{modulePrefix}:enum.TemplateStatus.DISABLE"),
    /** 预发布（发布前的审核中间态，业务上介于草稿与发布之间） */
    PREPUBISH(3, "{modulePrefix}:enum.TemplateStatus.PREPUBISH");

    /** 数据库存储整型值 */
    private Integer value;
    /** 国际化消息 key，格式：{modulePrefix}:enum.TemplateStatus.{枚举名} */
    private String messageKey;

    @Override
    public Integer getValue() {
        return value;
    }

    @Override
    public String getMessageKey() {
        return messageKey;
    }

    /**
     * JPA 枚举转换器：数据库存整型（0/1/2/3），而非枚举名称字符串
     * 在 Entity 字段上使用 @Convert(converter = TemplateStatus.Converter.class)
     */
    public static class Converter extends IEnum.Converter<Integer, TemplateStatus> {
    }
}
```

---

## 步骤1：添加字段到模板实体 Entity

**核心要点**：
- `fdCategory`：字段名必须为 `fdCategory`，使用 `@ManyToOne` + `@JoinColumn`，`{Module}CategoryUtil` 的查询路径（如 `fdCategory.fdHierarchyId`）依赖此名称
- `fdStatus`：使用 `@Convert(converter = TemplateStatus.Converter.class)` 而非 `@Enumerated`，数据库存整型
- 模板实体标准接口：`FdName4Language, FdDesc4Language, FdOrder, FdLastModifiedTime, FdCreator, FdCreateTime, FdAlter, FdAlterTime`
- 如有内容体字段（富文本/JSON/HTML），必须使用 `@Lob` + `FetchType.LAZY`，避免列表查询加载大字段

| 接口/字段 | 包 | 说明 |
|-----------|----|------|
| `FdName4Language` | `com.landray.common.core.data.field` | 模板名称（多语言） |
| `FdDesc4Language` | `com.landray.common.core.data.field` | 模板描述（多语言） |
| `FdOrder` | `com.landray.common.core.data.field` | 排序号 |
| `FdLastModifiedTime` | `com.landray.common.core.data.field` | 最后修改时间 |
| `FdCreator` | `com.landray.sys.auth.data.field` | 创建人 |
| `FdCreateTime` | `com.landray.common.core.data.field` | 创建时间 |
| `FdAlter` | `com.landray.sys.auth.data.field` | 修改人 |
| `FdAlterTime` | `com.landray.common.core.data.field` | 修改时间 |

```java
// 本机制新建类的自模块导入：
import {modulePackage}.constant.TemplateStatus;
// 其他依赖包参见 ref-00-bundle-spec.md

@Getter
@Setter
@Entity
@Table
@MetaEntity(messageKey = "{modulePrefix}:table.{module}Template")
public class {Module}Template extends AbstractEntity
        implements FdName4Language, FdDesc4Language, FdOrder, FdLastModifiedTime,
                   FdCreator, FdCreateTime, FdAlter, FdAlterTime {

    /** 字段名必须为 fdCategory，框架查询路径（如 fdCategory.fdHierarchyId）依赖此名称 */
    @ManyToOne
    @JoinColumn(name = "fd_category_id")
    @MetaProperty(messageKey = "{modulePrefix}:{module}Template.fdCategory")
    private {Module}Category fdCategory;

    /** 使用 Converter 存整型，不使用 @Enumerated(EnumType.STRING) */
    @Convert(converter = TemplateStatus.Converter.class)
    @MetaProperty(messageKey = "{modulePrefix}:{module}Template.fdStatus")
    private TemplateStatus fdStatus;

    // 其他业务字段...
}
```

---

## 步骤2：创建模板 VO

**核心要点**：
- `fdCategory` 使用 `{Module}CategoryVO` 存储分类引用（只存 id+name，不展开完整对象）
- `fdStatus` 使用 `TemplateStatus` 枚举类型，序列化时框架自动处理枚举值
- 如有内容体字段，列表接口中应排除，详情接口才返回（避免大字段传输）

```java
// 本机制新建类的自模块导入：
import {modulePackage}.constant.TemplateStatus;
// 其他依赖包参见 ref-00-bundle-spec.md

@Getter
@Setter
@ApiModel("{module}模板VO")
public class {Module}TemplateVO extends AbstractVO {

    @ApiModelProperty("模板名称")
    private String fdName;

    @ApiModelProperty("排序号")
    private Integer fdOrder;

    @ApiModelProperty("模板状态（DRAFT/PREPUBISH/PUBLISH/DISABLE）")
    private TemplateStatus fdStatus;

    @ApiModelProperty("所属分类")
    private {Module}CategoryVO fdCategory;

    @ApiModelProperty("创建人")
    private IdNameProperty fdCreator;

    @ApiModelProperty("修改人")
    private IdNameProperty fdAlter;

    @ApiModelProperty("修改时间")
    private Date fdAlterTime;
}
```

---

## 步骤3：创建模板 API 接口

**核心要点**：
- `publish` 是模板特有的业务动作，区别于 `update`：`update` 仅做字段持久化
- `filterCategoryIdWithTemplate` 供 SearchService 通过 API 接口调用，不暴露 HTTP 端点（无 `@PostMapping`）

```java
@Api(tags = "{module}模板服务")
public interface I{Module}TemplateApi extends IApi<{Module}TemplateVO> {

    /**
     * 发布模板（DRAFT/PREPUBISH → PUBLISH）。
     * 与 update 的区别：publish 代表业务"生效"动作，可附加内容校验、事件触发等逻辑。
     */
    @PostMapping("publish")
    {Module}TemplateVO publish(@RequestBody {Module}TemplateVO templateVO);

    /**
     * 从候选分类 ID 中过滤出拥有已发布模板的分类 ID 集合。
     * 供 SearchService.getTreeInRun 回填 childFlag，不暴露 HTTP 端点。
     *
     * @param categoryIds 当前层分类 ID 列表
     * @return 其中含已发布模板的分类 ID 集合
     */
    @PostMapping("filterCategoryIdWithTemplate")
    Set<String> filterCategoryIdWithTemplate(List<String> categoryIds);
}
```

---

## 步骤4：创建模板 Repository

**核心要点**：
- `countByCategory` 和 `findAllByCategory` 是分类拷贝和删除校验的基础依赖，必须实现
- `countByFdName` 用于 `copy` 内部的名称去重

```java
@Repository
public interface I{Module}TemplateRepository extends IRepository<{Module}Template> {

    /**
     * 统计指定分类下的模板数量，供分类 delete 前置校验（有模板时禁止删除分类）。
     */
    @Query("select count(1) from {Module}Template where fdCategory.fdId = ?1")
    long countByCategory(String fdCategoryId);

    /**
     * 查询指定分类下的所有模板（用于 copyByCategory 批量复制）。
     */
    @Query("select t from {Module}Template t where fdCategory.fdId = ?1")
    List<{Module}Template> findAllByCategory(String fdCategoryId);

    /**
     * 统计同名模板数量，配合 BeanHelper.setAvailableValue 保证拷贝时名称唯一。
     */
    @Query("select count(1) from {Module}Template where fdName = ?1")
    long countByFdName(String fdName);

    /**
     * 从候选分类 ID 中过滤出拥有已发布模板的分类 ID 集合，
     * 供 getTreeInRun 回填 childFlag（有已发布模板的分类节点显示展开箭头）。
     */
    @Query("select distinct t.fdCategory.fdId from {Module}Template t " +
           "where t.fdCategory.fdId in ?1 and t.fdStatus = com.landray.{modulePrefix}.{moduleSuffix}.constant.TemplateStatus.PUBLISH")
    Set<String> filterCategoryIdWithTemplate(List<String> categoryIds);
}
```

---

## 步骤5：创建模板 Service

**核心要点**：
- 按需重写 `doInit`：初始化模板实体，设置默认状态为 `DRAFT`，并配置机制加载参数
- `publish`：**必须实现**（来自 `I{Module}TemplateApi`）；状态流转：`DRAFT` → `PUBLISH`，`PREPUBISH` → `PUBLISH`；调用 `update` 持久化后可触发业务事件
- `countByCategory`：供分类 Service 的 `delete` 前置校验调用，**必须以 `public` 方法暴露**
- `copyByCategory`：供分类 `copy` 递归时调用，**必须以 `public` 方法暴露**；内部用 `findAllByCategory` 批量查出该分类下所有模板，逐一执行拷贝
- `copy` 拷贝时必须将状态重置为 `DRAFT`，拷贝的模板不应直接继承原模板的发布状态

**状态流转说明**：

| 当前状态 | 触发动作 | 目标状态 | 说明 |
|---------|---------|---------|------|
| `DRAFT` | `publish` | `PUBLISH` | 草稿直接发布 |
| `PREPUBISH` | `publish` | `PUBLISH` | 预发布审核通过后发布 |
| `PUBLISH` | `publish` | `PUBLISH` | 已发布再次调用，状态不变（幂等） |
| `DISABLE` | `publish` | `PUBLISH` | 禁用后重新发布（按业务决定是否支持） |
| 任意 | `copy` | `DRAFT` | 拷贝的新模板始终从草稿状态开始 |

```java
// 本机制新建类的自模块导入：
import {modulePackage}.constant.TemplateStatus;
// 其他依赖包参见 ref-00-bundle-spec.md

@Slf4j
@Service
@RestController
@RequestMapping("/api/{modulePrefix}/{module}Template")
@Transactional(rollbackFor = Exception.class)
public class {Module}TemplateService
        extends AbstractServiceImpl<I{Module}TemplateRepository, {Module}Template, {Module}TemplateVO>
        implements I{Module}TemplateApi {

    /**
     * 初始化模板实体：默认状态 DRAFT，并设置 mechanisms.load="*" 确保 loadById 可加载全部机制数据。
     */
    @Override
    protected void doInit({Module}Template entity) {
        super.doInit(entity);
        // 新建模板默认为草稿状态
        if (entity.getFdStatus() == null) {
            entity.setFdStatus(TemplateStatus.DRAFT);
        }
        // 初始化机制加载配置（"*" 表示加载全部机制数据）
        if (entity.getMechanisms() == null) {
            entity.setMechanisms(new HashMap<>(4));
        }
        entity.getMechanisms().put("load", "*");
    }

    /** 保存/更新前处理，按需补充业务逻辑（自动填充字段、数据转换等）。 */
    @Override
    protected void beforeSaveOrUpdate({Module}Template entity, boolean isAdd) {
        super.beforeSaveOrUpdate(entity, isAdd);
    }

    /** 发布模板（DRAFT/PREPUBISH → PUBLISH），已是 PUBLISH 则幂等不变。 */
    @Override
    public {Module}TemplateVO publish({Module}TemplateVO templateVO) {
        // 仅从草稿或预发布状态推进到已发布，避免状态回退
        if (templateVO.getFdStatus() != TemplateStatus.PUBLISH) {
            templateVO.setFdStatus(TemplateStatus.PUBLISH);
        }
        update(templateVO);
        // 可在此发布事件，通知下游系统（如流程引擎、消息通知等）
        // applicationContext.publishEvent(new TemplatePublishEvent(templateVO));
        return templateVO;
    }

    /** 供分类 Service delete 前置校验调用（public，非接口方法）。 */
    public long countByCategory(String fdCategoryId) {
        return repository.countByCategory(fdCategoryId);
    }

    /** 供 SearchService.getTreeInRun 回填 childFlag，委托给 Repository 查询（实现 I{Module}TemplateApi 接口）。 */
    @Override
    public Set<String> filterCategoryIdWithTemplate(List<String> categoryIds) {
        return repository.filterCategoryIdWithTemplate(categoryIds);
    }

    /** 供分类 copy 递归时调用（public，非接口方法）。 */
    public void copyByCategory({Module}Category source, {Module}Category target) {
        List<{Module}Template> sourceList = repository.findAllByCategory(source.getFdId());
        if (!CollectionUtils.isEmpty(sourceList)) {
            sourceList.forEach(template -> copyTemplate(template, target));
        }
    }

    /**
     * 复制单个模板到目标分类。
     * 必须通过 mechanisms.put("load", "*") 加载完整机制数据后再拷贝，否则表单/流程/编号等配置会丢失。
     */
    private void copyTemplate({Module}Template source, {Module}Category targetCategory) {
        // 构造带机制加载标记的查询 VO，确保 loadById 返回完整机制数据
        IdVO idVO = IdVO.of(source.getFdId());
        idVO.setMechanisms(new HashMap<>(1));
        idVO.getMechanisms().put("load", "*");
        Optional<{Module}TemplateVO> sourceVOOpt = loadById(idVO);
        if (!sourceVOOpt.isPresent()) {
            return;
        }
        {Module}TemplateVO sourceVO = sourceVOOpt.get();
        // 将模板归属到目标分类（targetCategory 由 copyByCategory 保证非 null）
        {Module}CategoryVO categoryVO = new {Module}CategoryVO();
        categoryVO.setFdId(targetCategory.getFdId());
        categoryVO.setFdName(targetCategory.getFdName());
        sourceVO.setFdCategory(categoryVO);
        // 执行拷贝（含名称去重、状态重置为 DRAFT）
        copy(sourceVO);
    }

    /**
     * 模板拷贝核心逻辑：排除 fdId/mechanisms 后重新生成，fdStatus 重置为 DRAFT。
     * 若需复制表单/流程等机制数据，在 add(target) 前按机制类型逐一填充
     *（参考 EntityCopyUtil.copySysXFormDesignVO 等工具方法）。
     */
    protected {Module}TemplateVO copy({Module}TemplateVO source) {
        {Module}TemplateVO target = new {Module}TemplateVO();
        // 排除 fdId（下方重新生成）和 mechanisms（下方重新初始化，不沿用 source 引用）
        BeanUtils.copyProperties(source, target, "fdId", "mechanisms");
        target.setFdId(IDGenerator.generateID());
        // 拷贝的模板始终从草稿状态开始，需重新发布才能生效
        target.setFdStatus(TemplateStatus.DRAFT);
        BeanHelper.setAvailableValue(target, "fdName",
                name -> repository.countByFdName(name) == 0);
        target.setMechanisms(new HashMap<>(4));
        add(target);
        return target;
    }
}
```

---

> **树查询能力**（SearchService + Controller 树端点）参见 **[ref-03-template-tree-spec.md](ref-03-template-tree-spec.md)**。

---

## 步骤6：创建模板 Controller

**核心要点**：
- 实现 `CombineController`（= `MetaController` + `CrudController` + `DeleteAllController` + `ListController`），自动暴露完整 CRUD 端点
- 请求路径格式：`/data/{modulePrefix}/{module}Template`（前端直接调用此路径）

**`CombineController` 自动暴露的端点**（来自各子接口的 `default` 方法，无需手动实现）：

| 来源接口 | HTTP 端点 | 用途 | 权限分层建议 |
|---------|-----------|------|-------------|
| `MetaController` | `POST meta` | 读取实体元数据（字段列表、枚举等） | 登录即可访问 |
| `CrudController` | `POST init` | 打开新建页，获取初始化数据 | 需新建权限 |
| `CrudController` | `POST add` | 新增模板（状态默认 DRAFT） | 需新建权限 |
| `CrudController` | `POST get` | 获取模板详情 | 登录即可访问 |
| `CrudController` | `POST update` | 更新模板字段（不改变状态） | 需编辑权限 |
| `CrudController` | `POST delete` | 删除单条模板 | 需删除权限 |
| `DeleteAllController` | `POST deleteAll` | 批量删除模板 | 需删除权限 |
| `ListController` | `POST list` | 分页/条件查询模板列表 | 登录即可访问 |

**自定义端点**（需在本 Controller 中显式声明）：

| HTTP 端点 | 来源 | 说明 | 权限分层建议 |
|-----------|------|------|-------------|
| `POST publish` | `I{Module}TemplateApi#publish` | 发布模板（状态流转） | 需编辑权限 |

> 树查询端点（`getManagementTree` / `getRuntimeTree` / `treeSearch`）参见 **[ref-03-template-tree-spec.md](ref-03-template-tree-spec.md)** 步骤3。

**可能需要 `@Override` 的场景**：
- `list`：如需在列表查询前注入权限过滤条件（如只返回当前用户有权限的分类下的模板），需重写此方法
- `delete`：如需在删除前校验模板是否被主文档引用，需重写此方法

```java
// 其他依赖包参见 ref-00-bundle-spec.md

@RestController
@RequestMapping("/data/{modulePrefix}/{module}Template")
public class {Module}TemplateController
        extends AbstractController<I{Module}TemplateApi, {Module}TemplateVO>
        implements CombineController<I{Module}TemplateApi, {Module}TemplateVO> {

    /** 发布模板（DRAFT/PREPUBISH → PUBLISH），需编辑权限。 */
    @PostMapping("publish")
    public Response<{Module}TemplateVO> publish(@RequestBody {Module}TemplateVO templateVO) {
        return Response.ok(getApi().publish(templateVO));
    }
}
```

---

## 步骤7：配置国际化资源

```properties
# 模板实体表名国际化
table.{module}Template=模板

# 模板字段国际化
{module}Template.fdName=模板名称
{module}Template.fdCategory=所属分类
{module}Template.fdStatus=模板状态

# 模板状态枚举国际化
enum.TemplateStatus.DRAFT=草稿
enum.TemplateStatus.PUBLISH=已发布
enum.TemplateStatus.DISABLE=已禁用
enum.TemplateStatus.PREPUBISH=预发布
```

---

## 注意事项

1. **`fdCategory` 字段名**：模板实体中关联分类的字段**必须**命名为 `fdCategory`，框架查询路径（`{Module}CategoryUtil`）依赖此名称
2. **`fdStatus` 使用 `@Convert` 而非 `@Enumerated`**：`@Convert(converter = TemplateStatus.Converter.class)` 将枚举存为整型（0/1/2/3）；不要使用 `@Enumerated(EnumType.STRING)` 或 `@Enumerated(EnumType.ORDINAL)`
3. **`TemplateStatus` 枚举拼写**：参考实现中预发布枚举值拼写为 `PREPUBISH`（无第二个 `L`），保持与实际代码一致，避免序列化不匹配
4. **`copy` 状态重置为 `DRAFT`**：拷贝的模板不能继承原模板的发布状态，必须从草稿重新走发布流程
5. **`countByCategory` 和 `copyByCategory` 是 public 方法而非接口方法**：分类 Service 直接注入模板 Service 并调用这两个方法，它们不在 `I{Module}TemplateApi` 中声明，必须以 `public` 方法暴露
6. **`copyByCategory` 需加载完整机制数据**：复制模板时需先用 `mechanisms.put("load", "*")` 加载机制配置，否则拷贝出的模板会丢失表单/流程/编号等机制数据
7. **内容体字段**：如需存储富文本/JSON 等大字段，必须使用 `@Lob` + `FetchType.LAZY`，并在 VO 中仅在详情接口返回
8. **`publish` 幂等性**：已处于 `PUBLISH` 状态的模板再次调用 `publish`，状态不变；避免将已发布的模板状态意外回退
9. **`QueryResult` 取列表**：`findAll(request)` 返回 `QueryResult`，取列表必须调用 `.getContent()`，禁止使用不存在的 `.getRows()`
10. **`addCondition` operator 参数**：必须使用 `QueryConstant.Operator` 枚举（如 `QueryConstant.Operator.eq`），禁止传入原始字符串（如 `"eq"`）
