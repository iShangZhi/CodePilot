# ref-03-composite-validator-spec — 复合嵌套鉴权规范

> 同时满足多组条件（AND of ORs）时使用 `@CompsiteValidators`，典型场景：既要有管理角色，又要有业务数据权限。

**核心规则**
- **结构**：外层 `@CompsiteValidators`（AND） → 内层每组 `@AuthValidators`（OR），形成"所有组都通过，组内任一通过"
- **仅用于方法级**：`@CompsiteValidators` 不能标在类上
- **覆盖类级别**：方法上的 `@CompsiteValidators` 完全替换类级别的 `@AuthValidators`，不叠加
- **`{Module}AuthConstant`**：使用自定义校验器时，用 `interface` 集中存放 `@ValidateParam(name=)` 的参数键名常量；与 `{Module}Role`（角色编码）职责不同，不要混用

**AND of ORs 复合校验**

```java
package {包路径}.core.controller;

import {包路径}.core.config.{Module}Role;
import {包路径}.core.auth.{Module}XxxValidator;
import {包路径}.core.auth.constant.{Module}AuthConstant;
import com.landray.sys.auth.constant.ValidatorOperator;
import com.landray.sys.auth.validator.annotation.AuthValidator;
import com.landray.sys.auth.validator.annotation.AuthValidators;
import com.landray.sys.auth.validator.annotation.CompsiteValidators;
import com.landray.sys.auth.validator.annotation.ValidateParam;

@RestController
@RequestMapping("/data/{模块Key}/{entity}")
// 类级别默认：SETTING 或 ADMIN 可访问
@AuthValidators(opt = ValidatorOperator.OPT_OR, value = {
    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_SETTING)}),
    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_ADMIN)})
})
public class {Entity}Controller extends AbstractController<I{Entity}Api, {Entity}VO> {

    // 方法级复合校验：同时满足"管理角色"和"业务数据权限"（覆盖类级别）
    @CompsiteValidators(opt = ValidatorOperator.OPT_AND, validators = {
            // 第一组 OR：必须有管理角色之一
            @AuthValidators(opt = ValidatorOperator.OPT_OR, value = {
                    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_SETTING)}),
                    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_ADMIN)})
            }),
            // 第二组 OR：必须满足业务数据权限之一
            @AuthValidators(opt = ValidatorOperator.OPT_OR, value = {
                    @AuthValidator(value = "roleValidator", params = {@ValidateParam({Module}Role.ROLE_{ROLE}_ADMIN)}),
                    @AuthValidator(value = "{ValidatorId1}", params = {
                            @ValidateParam(name = {Module}AuthConstant.PARAM_ID, value = "${vo.fdParent.fdId}")}),
                    @AuthValidator(value = "{ValidatorId2}", params = {
                            @ValidateParam(name = {Module}AuthConstant.PARAM_ID, value = "${vo.fdParent.fdId}")})
            })
    })
    @Override
    public Response<IdVO> add({Entity}VO vo) {
        return Response.ok(getApi().add(vo));
    }
}
```

**参数键名常量接口**

```java
package {包路径}.core.auth.constant;

/**
 * 权限校验器参数键名常量，供 @ValidateParam(name=) 引用
 * 注意：与 {Module}Role（角色编码常量）职责不同，不要混用
 */
public interface {Module}AuthConstant {
    String PARAM_TEMPLATE_ID = "templateId";   // 对应 @ValidateParam(name="templateId")
    String PARAM_CATEGORY_ID = "categoryId";   // 对应 @ValidateParam(name="categoryId")
    String PARAM_ID          = "fdId";         // 对应 @ValidateParam(name="fdId")
}
```

**易踩坑**
- **`@CompsiteValidators` 外层 `opt` 固定为 AND**：若需要 OR of ANDs，需重新设计校验器逻辑，注解层无法直接表达
- **`{Module}AuthConstant` vs `{Module}Role`**：前者是参数键名（`"fdId"`），后者是角色编码（`"ROLE_XX_ADMIN"`），两者都叫"权限常量"但用途完全不同
