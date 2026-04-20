# 主文档机制部署步骤（ref-04）

> 占位符含义参见主 SKILL 的**四、占位符说明**。
> 所有跨模块包导入以 [ref-00-bundle-spec.md](ref-00-bundle-spec.md) 为准；各步骤代码块中仅保留本机制新建类的导入。

---

## 步骤1：添加分类/模板字段到主文档实体 Entity

**核心要点**：
- 添加 `fdTemplate` 字段，使用 `@ManyToOne` + `@JoinColumn` 关联模板（主文档通过模板归属到分类）
- 主文档实体标准接口：`FdSubject, FdLastModifiedTime, FdCreator, FdCreateTime, FdAlter, FdAlterTime, FdCreatorDept, FdOwner, FdOwnerDept`

| 接口 | 包 | 说明 |
|------|----|------|
| `FdSubject` | `com.landray.common.core.data.field` | 标题（多语言用 `FdSubject4Language`） |
| `FdDelete` | `com.landray.common.core.data.field` | 删除标志 |
| `FdLastModifiedTime` | `com.landray.common.core.data.field` | 最后修改时间 |
| `FdCreator` | `com.landray.sys.auth.data.field` | 创建人 |
| `FdCreateTime` | `com.landray.common.core.data.field` | 创建时间 |
| `FdAlter` | `com.landray.sys.auth.data.field` | 修改人 |
| `FdAlterTime` | `com.landray.common.core.data.field` | 修改时间 |
| `FdCreatorDept` | `com.landray.sys.auth.data.field` | 创建人部门 |
| `FdOwner` | `com.landray.sys.auth.data.field` | 归属人 |
| `FdOwnerDept` | `com.landray.sys.auth.data.field` | 归属部门 |

```java
// 本机制无新建类专属导入；所有依赖包参见 ref-00-bundle-spec.md

@Getter
@Setter
@Entity
@Table
@MetaEntity(messageKey = "{modulePrefix}:table.{module}Main")
public class {Module}Main extends AbstractEntity
        implements FdSubject, FdLastModifiedTime,
                   FdCreator, FdCreateTime, FdAlter, FdAlterTime,
                   FdCreatorDept, FdOwner, FdOwnerDept {

    /**
     * 关联的模板（通过模板间接归属到分类）
     */
    @ManyToOne
    @JoinColumn(name = "fd_template_id")
    @MetaProperty(messageKey = "{modulePrefix}:{module}Main.fdTemplate")
    private {Module}Template fdTemplate;

    // 其他业务字段...
}
```

---

## 步骤2：创建主文档 VO

```java
@Getter
@Setter
@ApiModel("{module}主文档VO")
public class {Module}MainVO extends AbstractVO {

    @ApiModelProperty("标题")
    private String fdSubject;

    @ApiModelProperty("所属模板")
    private IdNameProperty fdTemplate;

    @ApiModelProperty("创建人")
    private IdNameProperty fdCreator;

    @ApiModelProperty("创建时间")
    private Date fdCreateTime;

    @ApiModelProperty("归属人")
    private IdNameProperty fdOwner;

    @ApiModelProperty("归属部门")
    private IdNameProperty fdOwnerDept;
}
```

---

## 步骤3：创建主文档 API 接口

```java
/**
 * {Module} 主文档服务接口，继承 {@link IApi} 获得标准 CRUD 声明，按需追加业务方法。
 */
@Api(tags = "{module}主文档服务")
public interface I{Module}MainApi extends IApi<{Module}MainVO> {
    // 按需扩展业务方法，例如：按模板统计文档数、按分类批量归档等
}
```

---

## 步骤4：创建主文档 Repository

```java
/**
 * {Module} 主文档数据访问接口，继承 {@link IRepository} 获得基础 CRUD，按需追加自定义查询。
 */
@Repository
public interface I{Module}MainRepository extends IRepository<{Module}Main> {
    // 按需扩展查询方法，例如：
    // @Query("select count(1) from {Module}Main where fdTemplate.fdId = ?1")
    // long countByTemplate(String templateId);
}
```

---

## 步骤5：创建主文档 Service

**核心要点**：
- `doPreload`：详情加载时按需填充关联字段（如模板内容、分类路径等）
- `doValidate`：保存前业务校验（如标题唯一性、模板是否存在等）
- `beforeSaveOrUpdate`：保存前处理（如自动填充默认值、根据模板初始化内容等）

```java
/**
 * {Module} 主文档 Service，兼作 Feign 服务端（{@code @RestController}），路径 {@code /api/{modulePrefix}/{module}Main}。
 */
@Slf4j
@Service
@RestController
@RequestMapping("/api/{modulePrefix}/{module}Main")
@Transactional(rollbackFor = Exception.class)
public class {Module}MainService
        extends AbstractServiceImpl<I{Module}MainRepository, {Module}Main, {Module}MainVO>
        implements I{Module}MainApi {

    /**
     * 详情预加载：用 fdId 重新加载主文档，回填 fdTemplate（详情页需要模板信息渲染表单/流程配置）。
     */
    @Override
    protected void doPreload({Module}MainVO vo) {
        if (!StringUtils.isEmpty(vo.getFdId())) {
            super.loadById(IdVO.of(vo.getFdId())).ifPresent(e -> vo.setFdTemplate(e.getFdTemplate()));
        }
        super.doPreload(vo);
    }
}
```

---

## 步骤6：创建主文档 Controller

**核心要点**：
- 同时实现 `PreloadController` + `CombineController`，在完整 CRUD 基础上额外暴露 `preload` 端点
- `PreloadController` 的 `preload` 端点用于详情页打开时填充关联数据（如模板内容、分类路径等），与 `CrudController` 的 `get` 分工：`get` 返回基础字段，`preload` 在此基础上做额外关联数据填充
- 请求路径格式：`/data/{modulePrefix}/{module}Main`

**自动暴露的端点**：

| 来源接口 | HTTP 端点 | 用途 | 权限分层建议 |
|---------|-----------|------|-------------|
| `MetaController` | `POST meta` | 读取实体元数据 | 登录即可访问 |
| `CrudController` | `POST init` | 打开新建页，获取初始化数据 | 需新建权限 |
| `CrudController` | `POST add` | 新增主文档 | 需新建权限 |
| `CrudController` | `POST get` | 获取主文档基础详情 | 需查看权限 |
| `CrudController` | `POST update` | 更新主文档 | 需编辑权限 |
| `CrudController` | `POST delete` | 删除单条主文档 | 需删除权限 |
| `DeleteAllController` | `POST deleteAll` | 批量删除主文档 | 需删除权限 |
| `ListController` | `POST list` | 分页/条件查询主文档列表 | 需查看权限 |
| `PreloadController` | `POST preload` | 详情页预加载，填充关联数据 | 需查看权限 |

**需要 `@Override` 的默认方法**：
- `preload`（来自 `PreloadController`）：`PreloadController` 的 `default preload` 直接调用 `getApi().preload(vo)`，若 Service 中 `preload` 有自定义填充逻辑则自动生效；但若 Controller 层需要在返回前做额外处理（如转换格式、脱敏），需重写此方法
- `list`：如需按当前用户权限过滤主文档列表，需重写此方法注入权限条件

```java
/**
 * {Module} 主文档 Controller，路径 {@code /data/{modulePrefix}/{module}Main}，
 * 实现 {@link CombineController} 暴露完整 CRUD，实现 {@link PreloadController} 暴露 {@code preload} 端点。
 */
@RestController
@RequestMapping("/data/{modulePrefix}/{module}Main")
public class {Module}MainController
        extends AbstractController<I{Module}MainApi, {Module}MainVO>
        implements PreloadController<I{Module}MainApi, {Module}MainVO>,
                   CombineController<I{Module}MainApi, {Module}MainVO> {

    /**
     * 重写预加载端点（POST preload）。
     * default 实现已调用 getApi().preload(vo)，如无 Controller 层额外处理可删除此 @Override。
     */
    @Override
    public Response<{Module}MainVO> preload(@RequestBody {Module}MainVO vo) {
        return PreloadController.super.preload(vo);
    }
}
```

---

## 步骤7：配置国际化资源

```properties
# 主文档实体表名国际化
table.{module}Main=主文档

# 主文档字段国际化
{module}Main.fdSubject=标题
{module}Main.fdTemplate=所属模板
```

---

## 按分类查询主文档

通过 `{Module}CategoryUtil.swapHierarchyId` 可以实现按分类（含子分类）查询主文档：

```java
// 按分类查询主文档（通过模板中转关联）
QueryRequest request = new QueryRequest();
request.addCondition("fdTemplate.fdCategory.fdHierarchyId",
    QueryConstant.Operator.child, "categoryId");

{Module}CategoryUtil.swapHierarchyId(request, "fdTemplate.fdCategory.fdHierarchyId");

QueryResult<{Module}MainVO> result = mainApi.findAll(request);
```
