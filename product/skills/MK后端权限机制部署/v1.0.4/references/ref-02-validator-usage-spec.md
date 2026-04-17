# ref-02-validator-usage-spec — 简单鉴权规范

> 在 Controller 类或方法上用内置 `roleValidator` 进行角色校验，是最常见的鉴权方式。

**核心规则**
- **`roleValidator` 的 `@ValidateParam` 直接写角色编码，无 `name` 属性**：与自定义校验器的 `@ValidateParam(name="xxx", value="...")` 格式不同
- **类级别 vs 方法级别**：`@AuthValidators` / `@ModuleValidator` 标在类上作全局默认；方法上单独标注时**完全覆盖**类级别，不叠加
- **`@AuthValidators` 与 `@ModuleValidator` 功能等价，属性名不同**：`@AuthValidators` 用 `value=`，`@ModuleValidator` 用 `validators=`；后者语义更强调"模块入口开关"，通常用在主文档 Controller
- **多角色用 `OPT_OR`**：任一满足即放行（常见：SETTING 或 ADMIN 均可访问）

**后台管理接口（类级别，SETTING 或 ADMIN 可访问）**

```java
package {包路径}.core.controller;

import {包路径}.core.config.{Module}Role;
import com.landray.sys.auth.constant.ValidatorOperator;
import com.landray.sys.auth.validator.annotation.AuthValidator;
import com.landray.sys.auth.validator.annotation.AuthValidators;
import com.landray.sys.auth.validator.annotation.ValidateParam;

@RestController
@RequestMapping("/data/{模块Key}/{entity}")
@AuthValidators(opt = ValidatorOperator.OPT_OR, value = {
    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_SETTING)}),  /* {模块语义}_后台管理 */
    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_ADMIN)})     /* {模块语义}_管理员 */
})
public class {Entity}Controller extends AbstractController<I{Entity}Api, {Entity}VO> {
    // 若某方法需要不同鉴权规则，在方法上单独标注 @AuthValidators 覆盖类级别
}
```

**主文档接口（@ModuleValidator，DEFAULT 或 ADMIN 可访问）**

```java
// @ModuleValidator 与 @AuthValidators 功能等价，主 Controller 惯用
@ModuleValidator(opt = ValidatorOperator.OPT_OR, validators = {
    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_DEFAULT)}),  /* {模块语义}_默认权限（所有登录用户）*/
    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_ADMIN)})     /* {模块语义}_管理员 */
})
public class {Module}MainController extends AbstractController<I{Module}MainApi, {Module}MainVO> {
}
```

**方法级覆盖（放宽权限，如只读接口对 DEFAULT 开放）**

```java
// 类级别为 SETTING/ADMIN，此方法单独覆盖为 DEFAULT 也可访问
@PostMapping("findById")
@AuthValidators(@AuthValidator(value = "roleValidator",
        params = {@ValidateParam({Module}Role.ROLE_{ROLE}_DEFAULT)}))  /* {模块语义}_默认权限 */
public Response<{Entity}VO> findById(@RequestBody IdVO idVO) {
    return Response.ok(getApi().findById(idVO));
}
```

**继承类级别权限的方法（无方法级注解）**

> 当方法不标注方法级鉴权注解时，自动继承类级别的 `@AuthValidators`。为提高可读性，须在方法上方添加注释说明权限来源。

```java
/**
 * 复制分类节点
 * @param idVO 源节点 ID
 * @return 新建节点 ID
 */
// 权限：继承类级别（SETTING/ADMIN）
@PostMapping("copy")
public Response<IdVO> copy(@RequestBody IdVO idVO) {
    return Response.ok(getApi().copy(idVO));
}
```

**Javadoc 完整性要求**

> Controller 中所有 `public` 方法**必须**有完整的 Javadoc 注释，包括方法用途描述。遗漏 Javadoc 是常见的 diff 偏差来源。

**易踩坑**
- **方法级覆盖是替换，不是追加**：若需要类级别 + 方法级双重校验，改用 `@CompsiteValidators`（见 ref-03）
- **`roleValidator` 与自定义校验器的 `@ValidateParam` 格式不同**：`roleValidator` 只填 `value`（角色编码）；自定义校验器还需填 `name`（参数键名）
- **继承类级别权限时要加注释**：方法不标注鉴权注解时权限来自类级别，但不写注释会让代码审阅者误以为遗漏，应显式标注 `// 权限：继承类级别（{角色列表}）`
