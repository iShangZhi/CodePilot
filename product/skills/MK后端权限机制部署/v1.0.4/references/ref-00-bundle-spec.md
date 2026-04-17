# ref-00-bundle-spec — 全局包导入规范

> 汇总权限机制 Skill 所有 ref 文件涉及的跨模块包导入。
> 所有 ref 文件中的代码块均须遵守此规范；ref 内代码只保留本类自模块的导入。

---

### JDK / 标准库

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
```

---

### Lombok

```java
import lombok.extern.slf4j.Slf4j;
```

---

### Hutool（按需）

```java
import cn.hutool.core.collection.CollUtil;
import cn.hutool.core.util.StrUtil;
```

---

### Spring Framework

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.util.CollectionUtils;
import org.springframework.util.StringUtils;
```

---

### 框架业务层 — 权限机制 API（mk-sys-auth-api）

```java
// 角色注解
import com.landray.sys.auth.role.annotation.AuthRole;
import com.landray.sys.auth.role.annotation.AuthRoles;

// 菜单角色常量（SETTING 角色继承用）
import com.landray.sys.auth.constant.menu.MenuRoleConstant;

// 校验器注解
import com.landray.sys.auth.validator.annotation.AuthValidator;
import com.landray.sys.auth.validator.annotation.AuthValidators;
import com.landray.sys.auth.validator.annotation.CompsiteValidators;
import com.landray.sys.auth.validator.annotation.ModuleValidator;
import com.landray.sys.auth.validator.annotation.ValidateParam;
import com.landray.sys.auth.validator.annotation.ValidatorDefine;

// 校验器运行时支持
import com.landray.sys.auth.constant.ValidatorOperator;
import com.landray.sys.auth.validator.support.AuthValidateContext;
import com.landray.sys.auth.validator.support.process.AbstractValidateProcess;

// 用户信息工具
import com.landray.sys.auth.util.UserUtil;
```

---

### Maven 依赖规则

**core 模块（必须）**

```xml
<!-- 角色注解、校验器注解、AbstractValidateProcess 均来自此依赖 -->
<dependency>
    <groupId>com.landray</groupId>
    <artifactId>mk-sys-auth-api</artifactId>
</dependency>
```

**api 模块（按需）**

```xml
<!-- 仅当 api 模块的 VO/DTO/基类中引用了 com.landray.sys.auth.* 下的类型时才声明 -->
<dependency>
    <groupId>com.landray</groupId>
    <artifactId>mk-sys-auth-api</artifactId>
</dependency>
```

> `version` 继承自平台 BOM，无需显式声明。
> **判断方法**：检查 api 模块 Java 源码中是否存在 `import com.landray.sys.auth.*` 的引用，有则声明，无则不声明。
