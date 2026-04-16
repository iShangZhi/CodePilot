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
ref-{序号}-{语义描述}-{后缀}.md
```

| 后缀 | 适用 | 示例 |
|------|------|------|
| `-spec` | 机制能力型 | `ref-01-category-spec.md` |
| `-integration` | 引擎集成型 | `ref-01-form-integration.md` |

序号 `00` 固定为 `conventions`，业务 ref 从 `01` 起。

## ref 拆分原则

| 情况 | 方式 |
|------|------|
| 有 Entity | 以 Entity 为切入点，覆盖 Entity → VO → API → Controller → Service → Repository |
| 无 Entity | 独立能力单独一个 ref（工具类 / 枚举 / 配置 / i18n） |

## ref 文件内部结构

每个代码单元按以下顺序组织：

1. **核心要点**（说明 + 表格）
2. 代码块（含行内注释，说明"为什么"而非"做什么"）
3. **注意事项**列表

## 元数据与内容规范

| 约束 | 说明 |
|------|------|
| SKILL.md frontmatter | 只含 `name` 和 `description`，不加其他字段 |
| 案例模块隔离 | references 中不出现案例模块的包名、类名、业务常量等具体内容 |
| 包导入规范 | ref-00-bundle-spec.md 集中声明跨模块包导入，按依赖层分组；所有其他 ref 只保留本机制新建类的自模块导入 |
| 注释粒度 | 接口方法/Service 公开方法须写 Javadoc；私有辅助方法须写 `/** */`；Entity/VO 非自解释字段须注明含义和约束 |
