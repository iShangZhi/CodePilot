# Plugin: maven-analyzer

通用 Maven 多模块工程信息提取能力。

## 能力描述

从任意 Maven 工程目录提取标准化模块元数据，不依赖具体业务框架。

## 提取规则

| 信息项 | 优先来源 | 备选来源 |
|--------|---------|---------|
| 模块名称（大驼峰） | 任意 Entity / Service 类名前缀 | `module.properties` → `moduleKey` |
| 模块 Key（kebab-case） | 根 `pom.xml` → `<artifactId>` | `module.properties` → `moduleKey` |
| 根目录名 | `{project}` 根目录名称 | — |
| 包路径 | 任意 `.java` 文件 `package` 声明 | `pom.xml` → `<groupId>` |
| 子模块列表 | 根 `pom.xml` → `<modules>` | 目录扫描 |
| 直接依赖 | 各子模块 `pom.xml` → `<dependencies>` | — |
| 继承基类 | `.java` 文件 `extends` 声明扫描 | — |
| 注解使用 | `.java` 文件注解扫描（`@` 前缀） | — |

## 输出格式

```markdown
### Maven 模块信息
- 模块名称：{Module}
- 模块 Key：{模块Key}
- 根目录：{根目录名}
- 包路径：{包路径}
- 子模块：{-api} / {-core} / {-client}（列举实际存在的）

### 依赖摘要
- 直接依赖：（列出 groupId:artifactId，去掉版本号）
- 继承基类：（类名 → 所在子模块）
- 主要注解：（注解名 → 出现位置）
```

## 使用方 Agent 调用约定

```
调用 maven-analyzer，分析路径：{project_path}
输出模块信息摘要，写入上下文，不超过 50 行
```
