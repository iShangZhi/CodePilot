# ref-04-custom-validator-spec — 自定义校验器规范

> 内置 `roleValidator` 无法满足时，继承 `AbstractValidateProcess` 实现基于业务数据的动态权限判断。

**核心规则**
- **文件位置**：`{模块}-core/.../core/auth/{Module}XxxValidator.java`
- **`@Component` 条件性**：需要 `@Autowired` 注入 Service 时必须加；通过 `context.getApplicationContext().getBean()` 手动取 Bean 时不需要
- **`@ValidatorDefine(id)` 引用常量**：`id` 应引用本类的 `NAME` 常量，不硬编码字符串；`NAME` 的值在 `@AuthValidator(value=)` 中使用
- **参数为空时的返回值**：`doValidate()` 取不到业务参数时，按业务语义决定返回 `true`（宽松）或 `false`（严格），不要统一返回 `true`
- **抽象父类模式**：多个校验器共享参数提取逻辑时，抽取 abstract 父类；父类**不标注** `@ValidatorDefine`

**带 @Component（使用 @Autowired 注入 Service）**

```java
package {包路径}.core.auth;

import com.landray.sys.auth.util.UserUtil;
import com.landray.sys.auth.validator.annotation.ValidateParam;
import com.landray.sys.auth.validator.annotation.ValidatorDefine;
import com.landray.sys.auth.validator.support.AuthValidateContext;
import com.landray.sys.auth.validator.support.process.AbstractValidateProcess;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

/**
 * {Module} 自定义权限校验器 — {校验逻辑描述}
 * 注册 id：{@value NAME}，在 @AuthValidator(value=NAME) 中引用
 */
@ValidatorDefine(id = {Module}XxxValidator.NAME)
@Component  // 有 @Autowired 时必须加
@Slf4j
public class {Module}XxxValidator extends AbstractValidateProcess {

    /** 校验器注册 id，全系统唯一，命名规范：{module}XxxValidator */
    public static final String NAME = "{module}XxxValidator";
    /** @ValidateParam(name=) 中的参数键名 */
    public static final String FD_ID = "fdId";

    @Autowired
    private I{Entity}Api {entity}Api;

    @Override
    public boolean doValidate(AuthValidateContext context, ValidateParam[] params) {
        String fdId = this.getParamValue(context, params, FD_ID);
        if (!StringUtils.hasText(fdId)) {
            log.debug("[{Module}XxxValidator] fdId 为空，返回 true（宽松策略）");
            return true;  // 按业务语义决定：true=宽松 / false=严格
        }
        // return {entity}Api.hasPermission(fdId, UserUtil.getCurUserInfo().getFdId());
        return true;
    }
}
```

**不带 @Component（通过 ApplicationContext 手动取 Bean）**

```java
import com.landray.sys.auth.validator.annotation.ValidateParam;
import com.landray.sys.auth.validator.annotation.ValidatorDefine;
import com.landray.sys.auth.validator.support.AuthValidateContext;
import com.landray.sys.auth.validator.support.process.AbstractValidateProcess;
import lombok.extern.slf4j.Slf4j;
import org.springframework.util.StringUtils;

@ValidatorDefine(id = {Module}AnotherValidator.NAME)
@Slf4j
public class {Module}AnotherValidator extends AbstractValidateProcess {

    public static final String NAME = "{module}AnotherValidator";
    public static final String MAIN_ID = "mainId";

    @Override
    public boolean doValidate(AuthValidateContext context, ValidateParam[] params) {
        String mainId = this.getParamValue(context, params, MAIN_ID);
        if (!StringUtils.hasText(mainId)) {
            return true;
        }
        // 无 @Component 时通过 ApplicationContext 手动取 Bean
        I{Entity}Api api = context.getApplicationContext().getBean(I{Entity}Api.class);
        // return api.checkXxx(mainId);
        return true;
    }
}
```

**抽象父类模式（多个校验器共享参数提取逻辑）**

```java
// 抽象父类：不标 @ValidatorDefine，只封装公共参数提取
@Slf4j
public abstract class Abstract{Module}XxxValidator extends AbstractValidateProcess {

    public static final String FD_ID = "fdId";
    public static final String PARENT_ID = "fdParentId";

    /** 先取 fdId，为空再取 fdParentId */
    protected String getIdOrParentId(AuthValidateContext context, ValidateParam[] params) {
        String id = this.getParamValue(context, params, FD_ID);
        if (StrUtil.isBlank(id)) {
            id = this.getParamValue(context, params, PARENT_ID);
        }
        return id;
    }
}

// 子类只关注业务逻辑，参数提取复用父类
@ValidatorDefine(id = {Module}SubValidator.NAME)
@Component
@Slf4j
public class {Module}SubValidator extends Abstract{Module}XxxValidator {

    public static final String NAME = "{module}SubValidator";

    @Autowired
    private I{Entity}Api {entity}Api;

    @Override
    public boolean doValidate(AuthValidateContext context, ValidateParam[] params) {
        String id = getIdOrParentId(context, params);
        if (StrUtil.isBlank(id)) {
            return false;  // 严格策略：取不到 id 则拒绝
        }
        // return {entity}Api.checkXxx(id);
        return true;
    }
}
```

**开启结果缓存（`isCache()`）**

> 适用场景：`doValidate` 内有数据库查询，且同一请求链路内可能被多次调用（如列表接口逐条鉴权）。

```java
@ValidatorDefine(id = {Module}XxxValidator.NAME)
@Component
@Slf4j
public class {Module}XxxValidator extends AbstractValidateProcess {

    public static final String NAME = "{module}XxxValidator";

    /**
     * 开启 ThreadLocal 缓存，避免同一请求内重复查库。
     * 缓存 key = validator.value() + MD5(请求参数 + @ValidateParam 内容)，请求结束自动清除。
     */
    @Override
    public boolean isCache() {
        return true;
    }

    @Override
    public boolean doValidate(AuthValidateContext context, ValidateParam[] params) {
        // 正常实现，缓存由父类 doValidateWithCache() 透明处理
        return true;
    }
}
```

**易踩坑**
- **`@Component` 与 `@ValidatorDefine` 独立**：两者没有因果关系；`@ValidatorDefine` 负责注册 id，`@Component` 负责 Spring 管理，按需添加
- **`isCache()` 默认 false**：`doValidate` 有数据库查询且同一请求内可能多次调用时，重写为 `true` 开启 ThreadLocal 缓存，避免重复查库
- **缓存 key 包含请求参数**：不同参数值不会命中同一缓存，安全；但参数序列化失败时缓存退化为 toString()，注意幂等性
