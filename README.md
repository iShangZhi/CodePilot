# CodePilot Hub

Claude Agent 驱动的后端 Skill 自进化框架。**Skill 是唯一产物，代码生成是验证手段。**

两条主线：

| 主线 | 做什么 | 输入 | 输出 |
|------|--------|------|------|
| **Skill 提炼** | 从机制源码 + 参考工程逆向提炼可复用规范 | 机制源码、参考工程 | `product/skills/{机制名}/` |
| **代码生成** | 以最新 SKILL 生成后端 Java 代码，人工审查后对话修订 SKILL | `latest/SKILL.md` | 目标工程代码 + 更新后的 SKILL |

---

## 工程结构

```
CodePilot/
├── .claude/
│   ├── config.yml              ★ 只改这里的 3 个路径
│   ├── agents/                   子 Agent 提示词
│   │   ├── extract-agent.md      Skill 提炼（机制源码 → SKILL）
│   │   ├── generate-agent.md     代码生成（SKILL → Java 代码）
│   │   └── patch-agent.md        SKILL 修正（对话修订 → 新版本）
│   ├── plugins/
│   │   ├── maven-analyzer.md     Maven 多模块信息提取
│   │   └── version-manager.md    vX.Y.Z 目录 + latest 软链接管理
│   └── skills/
│       └── skill-gen-prompt.md   Skill 提炼三阶段提示词
│
├── workspace/input/
│   └── module.md               ★ 每次任务只填这里
│
└── product/skills/
    └── {机制名}/               ★ SKILL 归档仓库（核心产物）
        ├── latest -> vX.Y.Z/     软链接，始终指向最新版本
        ├── vX.Y.Z/
        │   ├── SKILL.md
        │   └── references/
        ├── loop_state.md         版本状态
        └── update_log.md         变更记录
```

---

## 快速开始

### 1. 配置路径

编辑 `.claude/config.yml`，修改 3 个路径：

```yaml
paths:
  target_project:    /your/path/mk-km-xxx       # 代码生成目标工程
  reference_project: /your/path/mk-km-review    # 参考工程（只读）
  skill_library:     /your/path/.trae/skills    # MK Skill 库（只读）
```

### 2. 填写任务

编辑 `workspace/input/module.md`，选择执行模式并填写对应内容。

### 3. 启动

```
启动 CodePilot
```

---

## 外部依赖

| 资源 | 用途 | 权限 |
|------|------|------|
| 目标工程 | 代码写入 | 读写 |
| 参考工程 | Skill 提炼案例来源 | 只读 |
| MK Skill 库 | 规范参考（提炼时） | 只读 |
| 机制源码 | Skill 提炼时分析机制能力 | 只读 |
