# ref-00 命名与结构约定

## 占位符

| 占位符 | 说明 | 示例 |
|--------|------|------|
| `{Module}` | 类名前缀，大驼峰 | `KmNews` |
| `{module}` | 实体名，小驼峰 | `kmNews` |
| `{模块Key}` | 模块标识，小写+连字符 | `km-news` |
| `{modulePrefix}` | 包名前缀 | `km` |
| `{moduleSuffix}` | 包名后缀 | `news` |

---

## ref 文件命名

```
ref-{序号}-{语义描述}.md
```

序号 `00` 固定为 `bundle-spec`（跨模块包导入规范），业务 ref 从 `01` 起。

**部署 SKILL** 中的 ref 后缀：

| 后缀 | 适用 | 示例 |
|------|------|------|
| `-spec` | 机制能力型 | `ref-01-category-spec.md` |
| `-integration` | 引擎集成型 | `ref-01-form-integration.md` |

> 部署 SKILL 不维护自己的 ref-00，直接引用对应服务清单 SKILL 的 ref-00-bundle-spec.md

**服务清单 SKILL** 中的 ref 后缀：

| 后缀 | 适用 | 示例 |
|------|------|------|
| `-guide` | 端点说明型 | `ref-01-category-guide.md` |

---

## ref 拆分原则

**部署 SKILL**：

| 情况 | 方式 |
|------|------|
| 有 Entity | 以 Entity 为切入点，覆盖 Entity → VO → API（框架接口声明）→ Controller（框架 default 方法）→ Service（框架方法实现）→ Repository |
| 无 Entity | 独立能力单独一个 ref（工具类 / 枚举 / 配置 / i18n） |

**服务清单 SKILL**：

| 维度 | 方式 |
|------|------|
| 以实体域为第一维度 | 每个核心实体一个 `-guide` ref，只含该实体的 `/data/` 端点清单（接口路径 + 权限 + 使用场景） |

---

## ref 文件内部结构

**部署 SKILL 的 ref**（侧重"怎么接入框架"）：

1. **核心规则**（决策型：选什么、为什么、限制条件）
2. 按层次（Entity/VO/Service/Controller）组织代码骨架，注释说明"为什么这样继承/实现"
3. **易踩坑**列表

**服务清单 SKILL 的 ref**（侧重"/data/ 接口清单"）：

1. **统一规则**（请求方式、类级权限、Content-Type、响应格式）
2. **端点一览表**（接口完整路径 + 一句话说明）
3. **详细设计**（每个端点：权限 + 使用场景 + 注意事项）
4. **易踩坑**列表

---

## 元数据与内容规范

| 约束 | 说明 |
|------|------|
| SKILL.md frontmatter | 只含 `name` 和 `description`，不加其他字段 |
| 案例模块隔离 | references 中不出现案例模块的包名、类名、业务常量等具体内容 |
| 包导入规范 | 服务清单的 ref-00-bundle-spec.md 集中声明跨模块包导入，按依赖层分组；部署 SKILL 的各 ref 只写「参见「{机制名称}服务清单」ref-00-bundle-spec.md」 |
| 注释粒度 | 代码注释说明"为什么"而非"做什么"；接口方法/Service 公开方法须写 Javadoc |
| `@OperLog` 禁止写入 | 属于业务个性化配置，SKILL 模板一律省略 |
