---
name: MK后端权限机制部署
description: 为 MK 后端模块接入权限机制。分两阶段：初始化阶段仅需角色定义类；鉴权注解、复合校验、自定义校验器按业务场景或其他机制的需要动态追加。
---

## 一、能力概览

| 能力 | 核心类/注解 | 阶段 | 触发条件 |
|------|-----------|------|---------|
| 角色定义类 | `{Module}Role`（class，同时承担**角色常量容器** + `@AuthRoles` 注册） | **初始化（必须）** | 模块需要角色管理时，一次性建立 |
| 简单鉴权 | `@AuthValidators` + 内置 `roleValidator` | 按需追加 | 某个 Controller 需要接口级鉴权时 |
| 复合嵌套鉴权 | `@CompsiteValidators`（AND of ORs） | 按需追加 | 同时需要角色校验 + 业务数据权限校验时 |
| 自定义校验器 | `@ValidatorDefine` + `AbstractValidateProcess` | 按需追加 | 内置 `roleValidator` 无法满足动态权限判断时 |

**关键约束**
- `{Module}Role` 是模块唯一的"角色常量容器"，`public static final String` 常量与 `@AuthRoles` 定义在**同一个 class** 中，不拆分为 interface，不单独建常量类
- 内置 `roleValidator` 的 `@ValidateParam` 直接写角色编码，**无 `name` 属性**（与自定义校验器不同）
- `@CompsiteValidators` 只能标在方法上，且会完全覆盖类级别的 `@AuthValidators`

> **关于 `AuthConstant`**：`AuthConstant` 是平台内置的权限过滤常量接口（`AUTH_QUERYFILTER`、`SYS_READER` 等），业务模块**无需引用或继承**。模块自己的角色编码常量统一放在 `{Module}Role` 类中。

---

## 二、部署流程

### 阶段一 · 初始化（必须，每个模块只做一次）

**Maven 依赖**

| 模块 | 是否需要 `mk-sys-auth-api` | 判断规则 |
|------|---------------------------|---------|
| `{模块}-core` | **必须** | 角色注解、校验器注解、`AbstractValidateProcess` 均来自此依赖 |
| `{模块}-api` | **按需** | 若 api 模块的 VO/DTO/基类中引用了 `mk-sys-auth-api` 中的类型（如权限相关 DTO、`AuthRole` 等注解），则需引入；否则不引入 |
| `{模块}-client` | 不需要 | 客户端封装不涉及权限注解 |

> **判断方法**：检查 api 模块的 Java 源码中是否存在 `import com.landray.sys.auth.*` 的引用。有引用则在 api/pom.xml 中声明依赖，无引用则不声明。

```xml
<!-- core/pom.xml（必须） -->
<dependency>
    <groupId>com.landray</groupId>
    <artifactId>mk-sys-auth-api</artifactId>
</dependency>
```

| 步骤 | 参考文件 | 说明 |
|------|---------|------|
| 0. 添加 Maven 依赖 | 见上方依赖规则 | core 模块必须，api 模块按需 |
| 1. 创建角色定义类 | [ref-01-role-define-spec.md](references/ref-01-role-define-spec.md) | `{Module}Role.java`，class 形式，含角色常量 + `@AuthRoles` |

---

### 阶段二 · 按需追加（由业务场景或其他机制触发）

以下能力相互独立，按实际需求单独接入，不必全部实现：

| 能力 | 参考文件 | 何时追加 |
|------|---------|---------|
| A. Controller 鉴权 | [ref-02-validator-usage-spec.md](references/ref-02-validator-usage-spec.md) | 新增 Controller 需要角色校验时 |
| B. 复合嵌套鉴权 | [ref-03-composite-validator-spec.md](references/ref-03-composite-validator-spec.md) | 某个接口同时需要角色 + 业务数据双重校验时 |
| C. 自定义校验器 | [ref-04-custom-validator-spec.md](references/ref-04-custom-validator-spec.md) | 内置 `roleValidator` 无法满足时，新增一个校验器文件 |

> **典型追加路径**：其他机制（如分类模板、审批流）接入时，在对应 Controller 上追加 `@AuthValidators` 或 `@CompsiteValidators`，无需改动角色定义类。

---

## 四、部署验证清单

**阶段一（初始化，必查）**
- [ ] `{Module}Role` 类同时包含角色常量（`public static final String`）和 `@AuthRoles` 注解
- [ ] 每个 `@AuthRole` 的 `messageKey` / `desc` 格式：`{模块Key}:role.{ROLE_CODE}.name/desc`
- [ ] SETTING 角色的 `extendRoles` 继承 `MenuRoleConstant.MENU_SYS_APPLICATION_ROLE`
- [ ] ADMIN 角色的 `extendRoles` 包含 SETTING 及所有业务角色（超集）
- [ ] Maven 依赖：core 模块必须声明 `mk-sys-auth-api`；api 模块仅在有 `com.landray.sys.auth.*` 引用时声明

**能力 A：Controller 鉴权（用到时查）**
- [ ] `@ValidateParam` 无 `name` 属性，`value` 直接是角色编码字符串
- [ ] 所有 public 方法均有完整 Javadoc 注释
- [ ] 继承类级别权限的方法，若无方法级注解，须在方法上方添加注释说明权限来源（如 `// 权限：继承类级别（SETTING/ADMIN）`）

**能力 B：复合嵌套鉴权（用到时查）**
- [ ] `@CompsiteValidators` 外层 `opt` 为 AND，内层每组 `@AuthValidators` 的 `opt` 为 OR
- [ ] `@CompsiteValidators` 标在方法上，不在类上

**能力 C：自定义校验器（用到时查）**
- [ ] 有 `@Autowired` 则必须加 `@Component`；用 `getApplicationContext().getBean()` 则无需
- [ ] `@ValidatorDefine.id` 与 Controller 中 `@AuthValidator.value()` 字符串完全一致
- [ ] `getParamValue` 的 `name` 参数与 `@ValidateParam.name()` 大小写一致

---

## 五、代码注释规范

| 代码位置 | 注释要求 |
|---------|---------|
| `{Module}Role` 类 | 类 Javadoc 说明"权限角色定义"；每个常量行内注释说明语义 |
| `@AuthValidators` / `@CompsiteValidators` 每项 | 行内注释 `/* {模块语义}_{角色语义} */` |
| 自定义校验器类 | 类 Javadoc 说明校验逻辑；参数常量注释对应的 `@ValidateParam.name` |
| Controller 所有 public 方法 | **必须有完整 Javadoc**（方法用途、参数说明、返回值说明） |
| 继承类级别权限的方法 | 若方法本身不标注鉴权注解，须在方法上方添加行内注释说明权限继承关系，格式：`// 权限：继承类级别（{角色列表}）` |

---

## 六、占位符说明

| 占位符 | 含义 | 示例 |
|--------|------|------|
| `{Module}` | 模块名（大驼峰） | `KmNews` |
| `{module}` | 模块名（小驼峰） | `kmNews` |
| `{模块Key}` | 模块标识（kebab-case） | `km-news` |
| `{包路径}` | Java 包路径 | `com.landray.km.news` |
| `{ROLE}` | 角色前缀（全大写） | `KMNEWS` |
| `{ValidatorId}` | 自定义校验器注册 id（小驼峰） | `kmNewsXxxValidator` |
| `{paramName}` | `@ValidateParam.name()` 参数名 | `fdId` |
