# ref-01-role-define-spec — 角色定义规范

> 为模块向平台注册角色，并统一管理角色编码常量。

## 职责说明

`{Module}Role` 类同时承担两个职责：

| 职责 | 内容 |
|------|------|
| **角色常量容器** | `public static final String` 常量，供 `@AuthRoles`、`@ValidateParam`、业务代码统一引用 |
| **角色注册** | `@AuthRoles` 注解，向平台声明本模块拥有哪些角色 |

> **不要**另建 `{Module}AuthConstant` 或 `{Module}RoleConstant` 接口来存放角色编码。两职责必须合一，否则注解与常量产生隐式耦合，难以维护。

## 三个必选角色

| 角色后缀 | 语义 | extendRoles |
|---------|------|------------|
| `DEFAULT` | 所有已登录用户自动拥有，无需手动授权 | 无 |
| `SETTING` | 后台管理员，获得应用后台菜单入口 | `MenuRoleConstant.MENU_SYS_APPLICATION_ROLE` |
| `ADMIN` | 继承所有角色，是权限超集；新增业务角色时必须同步加入其 `extendRoles` | SETTING + DEFAULT + 所有业务角色 |

**核心规则**
- **文件**：`{模块}-core/.../core/config/{Module}Role.java`，`public class`（不得用 `interface` 或 `abstract class`）
- **常量与注解同类**：通过 `static import` 交叉引用，不拆分
- **单一原则**：一个模块只有**一个** `{Module}Role` 类，新增角色在此类中追加
- **常量命名**：`ROLE_{ROLE}_{TYPE}`，全大写；i18n key 格式：`{模块Key}:role.{ROLE_CODE}.name/desc`

**角色定义类**

```java
package {包路径}.core.config;

import com.landray.sys.auth.constant.menu.MenuRoleConstant;
import com.landray.sys.auth.role.annotation.AuthRole;
import com.landray.sys.auth.role.annotation.AuthRoles;

import static {包路径}.core.config.{Module}Role.*;  // static import 使 @AuthRoles 内可直接引用本类常量

/**
 * 权限角色定义 — 常量容器 + 角色注册，两职责合一
 */
@AuthRoles(roles = {
        // DEFAULT：所有已登录用户自动拥有，无需手动授权
        @AuthRole(name = ROLE_{ROLE}_DEFAULT,
                messageKey = "{模块Key}:role." + ROLE_{ROLE}_DEFAULT + ".name",
                desc = "{模块Key}:role." + ROLE_{ROLE}_DEFAULT + ".desc"),

        // ADMIN：继承所有业务角色，是权限超集；新增业务角色时必须同步加入 extendRoles
        @AuthRole(name = ROLE_{ROLE}_ADMIN,
                messageKey = "{模块Key}:role." + ROLE_{ROLE}_ADMIN + ".name",
                desc = "{模块Key}:role." + ROLE_{ROLE}_ADMIN + ".desc",
                extendRoles = {
                        ROLE_{ROLE}_SETTING,
                        ROLE_{ROLE}_DEFAULT,
                        ROLE_{ROLE}_MAIN_CREATE,
                        ROLE_{ROLE}_MAIN_READER,
                        ROLE_{ROLE}_MAIN_UPDATE,
                        ROLE_{ROLE}_MAIN_DELETE
                }),

        // SETTING：后台管理员，继承 APPLICATION_ROLE 获得应用后台菜单入口
        @AuthRole(name = ROLE_{ROLE}_SETTING,
                messageKey = "{模块Key}:role." + ROLE_{ROLE}_SETTING + ".name",
                desc = "{模块Key}:role." + ROLE_{ROLE}_SETTING + ".desc",
                extendRoles = MenuRoleConstant.MENU_SYS_APPLICATION_ROLE),

        // 业务角色按实际需求增减，示例如下
        @AuthRole(name = ROLE_{ROLE}_MAIN_CREATE,
                messageKey = "{模块Key}:role." + ROLE_{ROLE}_MAIN_CREATE + ".name",
                desc = "{模块Key}:role." + ROLE_{ROLE}_MAIN_CREATE + ".desc"),

        @AuthRole(name = ROLE_{ROLE}_MAIN_READER,
                messageKey = "{模块Key}:role." + ROLE_{ROLE}_MAIN_READER + ".name",
                desc = "{模块Key}:role." + ROLE_{ROLE}_MAIN_READER + ".desc"),

        @AuthRole(name = ROLE_{ROLE}_MAIN_UPDATE,
                messageKey = "{模块Key}:role." + ROLE_{ROLE}_MAIN_UPDATE + ".name",
                desc = "{模块Key}:role." + ROLE_{ROLE}_MAIN_UPDATE + ".desc"),

        @AuthRole(name = ROLE_{ROLE}_MAIN_DELETE,
                messageKey = "{模块Key}:role." + ROLE_{ROLE}_MAIN_DELETE + ".name",
                desc = "{模块Key}:role." + ROLE_{ROLE}_MAIN_DELETE + ".desc"),
})
public class {Module}Role {

    /** {模块语义}_默认权限 */
    public static final String ROLE_{ROLE}_DEFAULT = "ROLE_{ROLE}_DEFAULT";
    /** {模块语义}_管理员 */
    public static final String ROLE_{ROLE}_ADMIN   = "ROLE_{ROLE}_ADMIN";
    /** {模块语义}_后台管理 */
    public static final String ROLE_{ROLE}_SETTING = "ROLE_{ROLE}_SETTING";
    /** {模块语义}_创建文档 */
    public static final String ROLE_{ROLE}_MAIN_CREATE = "ROLE_{ROLE}_MAIN_CREATE";
    /** {模块语义}_阅读文档 */
    public static final String ROLE_{ROLE}_MAIN_READER = "ROLE_{ROLE}_MAIN_READER";
    /** {模块语义}_编辑文档 */
    public static final String ROLE_{ROLE}_MAIN_UPDATE = "ROLE_{ROLE}_MAIN_UPDATE";
    /** {模块语义}_删除文档 */
    public static final String ROLE_{ROLE}_MAIN_DELETE = "ROLE_{ROLE}_MAIN_DELETE";
}
```

**多语言配置（ApplicationResources.properties）**

> 文件位置：`{模块}-core/.../resource/ApplicationResources.properties`
> `messageKey` 中冒号前的模块命名空间（`{模块Key}:`）由平台自动剥离，properties key 只写冒号后的部分。

```properties
# 角色名称 / 描述 — 与 @AuthRole(messageKey/desc) 中冒号后的部分对应
role.ROLE_{ROLE}_DEFAULT.name={模块语义}_默认权限
role.ROLE_{ROLE}_DEFAULT.desc=访问"{模块语义}"模块的基本权限，没有该权限则不能访问整个模块
role.ROLE_{ROLE}_ADMIN.name={模块语义}_管理员
role.ROLE_{ROLE}_ADMIN.desc=允许对模块的所有功能进行操作
role.ROLE_{ROLE}_SETTING.name={模块语义}_后台管理
role.ROLE_{ROLE}_SETTING.desc=允许进入当前模块的后台
role.ROLE_{ROLE}_MAIN_CREATE.name={模块语义}_创建文档
role.ROLE_{ROLE}_MAIN_CREATE.desc=允许创建{模块语义}文档
role.ROLE_{ROLE}_MAIN_READER.name={模块语义}_阅读文档
role.ROLE_{ROLE}_MAIN_READER.desc=允许查阅所有{模块语义}文档
role.ROLE_{ROLE}_MAIN_UPDATE.name={模块语义}_编辑文档
role.ROLE_{ROLE}_MAIN_UPDATE.desc=允许编辑未结束的{模块语义}文档
role.ROLE_{ROLE}_MAIN_DELETE.name={模块语义}_删除文档
role.ROLE_{ROLE}_MAIN_DELETE.desc=允许删除所有{模块语义}文档
```

**易踩坑**
- **SETTING 用 `MENU_SYS_APPLICATION_ROLE`，不是 `MENU_SYS_MANAGER_ROLE`**：前者是应用级后台入口，后者是全局系统管理入口，误用会导致权限范围错误
- **角色编码上线后不可修改**：平台已持久化角色授权关系，改编码等于新建角色，旧授权全部失效
- **不要用 `AuthConstant`**：`AuthConstant` 是平台权限过滤常量（`AUTH_QUERYFILTER` 等），与角色编码无关，业务模块角色常量只放在 `{Module}Role` 中
