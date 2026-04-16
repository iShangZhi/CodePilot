# Plugin: version-manager

通用语义化版本目录管理能力（vX.Y.Z + latest 软链接 + 状态文件）。

## 能力描述

在指定目录下管理版本化子目录，维护 `latest` 软链接和 `loop_state.md` 状态文件。

## 版本号规则

| 变更类型 | 规则 | 示例 |
|---------|------|------|
| 首次创建 | `v1.0.0` | — |
| 内容小幅修正（patch） | patch +1 | `v1.0.1` → `v1.0.2` |
| 结构性重构（minor） | minor +1，patch 归零 | `v1.0.2` → `v1.1.0` |
| 不兼容重写（major） | major +1，其余归零 | `v1.1.0` → `v2.0.0` |

## 操作步骤

### 创建新版本目录

```bash
mkdir -p "{base_dir}/{version}"
```

### 更新 latest 软链接

```bash
# 删除旧软链接，创建指向新版本的软链接
rm -f "{base_dir}/latest"
ln -s "{version}" "{base_dir}/latest"
```

### 更新 loop_state.md

在 `{base_dir}/loop_state.md` 中更新以下字段：

```markdown
- 当前轮次：{N}
- 当前版本：{vX.Y.Z}
- 上轮偏差项数：{N}
- 本轮变更行数：{N}
- 下轮重点修正：{列表}
- 版本历史：追加一行 `{vX.Y.Z} — {YYYY-MM-DD} — {变更摘要}`
```

## 输出格式

```markdown
### 版本管理结果
- 新版本目录：{base_dir}/{vX.Y.Z}/  ✓
- latest 软链接：{base_dir}/latest → {vX.Y.Z}  ✓
- loop_state.md：已更新  ✓
```

## 使用方 Agent 调用约定

```
调用 version-manager，基础目录：{base_dir}，新版本号：{vX.Y.Z}
完成目录创建、软链接更新、loop_state 更新后返回确认
```
