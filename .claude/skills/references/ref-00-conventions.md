# ref-00 命名与结构约定

## 占位符

| 占位符 | 说明 | 示例 |
|--------|------|------|
| `{Module}` | 类名前缀，大驼峰 | `KmNews` |
| `{module}` | 实体名，小驼峰 | `kmNews` |
| `{模块Key}` | 模块标识，小写+连字符 | `km-news` |
| `{modulePrefix}` | 包名前缀 | `km` |
| `{moduleSuffix}` | 包名后缀 | `news` |

## ref 文件命名

```
ref-{序号}-{语义描述}.md
```

序号 `00` 固定为 `bundle-spec`（跨模块类型声明），业务 ref 从 `01` 起。

**部署 SKILL** 中的 ref 可选后缀：

| 后缀 | 适用 | 示例 |
|------|------|------|
| `-spec` | 机制能力型 | `ref-01-category-spec.md` |
| `-integration` | 引擎集成型 | `ref-01-form-integration.md` |

**服务清单 SKILL** 中的 ref 无固定后缀，按内容语义命名：

| 典型文件 | 内容 |
|---------|------|
| `ref-01-{实体名}-guide.md` | 实体外部可调用 API + 常用端点 + 注入调用示例 |
| `ref-02-service-guide.md` | 跨模块 Service public 方法 + 调用示例 |
| `ref-03-extension-guide.md` | 业务扩展点 + 实现模板（无则省略） |
| `ref-04-utils-guide.md` | 工具类 + 使用示例（无则省略） |

## ref 拆分原则

**部署 SKILL**：

| 情况 | 方式 |
|------|------|
| 有 Entity | 以 Entity 为切入点，覆盖 Entity → VO → API（框架接口声明）→ Controller（框架 default 方法）→ Service（框架方法实现）→ Repository |
| 无 Entity | 独立能力单独一个 ref（工具类 / 枚举 / 配置 / i18n） |

**服务清单 SKILL**：

| 维度 | 方式 |
|------|------|
| 以实体域为第一维度 | 每个核心实体一个 ref，含外部可调用 API（框架标准+自定义）和常用 /data/ 端点 |
| 横切关注点 | service-methods / extension-points / utils 各独立一个 ref |

## ref 文件内部结构

**部署 SKILL 的 ref**（侧重"怎么接入框架"）：

1. **核心规则**（决策型：选什么、为什么、限制条件）
2. 按层次（Entity/VO/Service/Controller）组织代码骨架，注释说明"为什么这样继承/实现"
3. **易踩坑**列表

**服务清单 SKILL 的 ref**（侧重"有什么、怎么用、怎么调"）：

1. **核心规则**（调用约束：参数要求、权限前提、异步/同步区别等）
2. 签名表格（方法/端点/扩展点清单）
3. **如何注入与调用**（注入代码 + 典型调用示例）
4. 按**业务场景**命名的代码示例（直接描述用途，不用"场景一/二/三"编号）
5. **易踩坑**列表

## 元数据与内容规范

| 约束 | 说明 |
|------|------|
| SKILL.md frontmatter | 只含 `name` 和 `description`，不加其他字段 |
| 案例模块隔离 | references 中不出现案例模块的包名、类名、业务常量等具体内容 |
| 包导入规范 | ref-00-bundle-spec.md 集中声明跨模块包导入，按依赖层分组；所有其他 ref 只保留本机制新建类的自模块导入 |
| 注释粒度 | 代码注释说明"为什么"而非"做什么"；接口方法/Service 公开方法须写 Javadoc |
| 服务清单调用示例 | 每个 ref 的"如何注入与调用"区块为必须项，不可省略 |
