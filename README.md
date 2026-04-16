# CodePilot Hub

以 **Skill 自进化**为核心目标的，一套以 Claude Agent 为引擎、代码生成为验证手段，让 Skill 在闭环迭代中自动收敛为高质量可复用规范的自进化工程框架。

通过「提炼 Skill → 生成代码验证 → 对比校验 → 反馈优化」的闭环迭代，驱动 Skill 持续收敛为高质量可复用规范。**后端代码生成是验证手段，Skill 是最终产物。**

---

## 工程产物

| 产物 | 性质 | 位置 |  用途 |
|------|------|------|------|
| **后端 Skill** | 核心产物 | `product/skill/{模块名}后端部署/` | 标准 MK Skill，`cp` 到 Skill 库后可在任意工程复用 |
| **验证代码记录** | 验证凭证 | `product/sample/{模块名}-vX.Y.Z/` | 证明 Skill 能产出符合规范的真实代码 |

---

## 总架构设计

```mermaid
flowchart TB
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% Layer 1 · 配置输入
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    subgraph L1["▌Layer 1 · 配置输入"]
        direction LR
        CFG["⚙ config.yml<br/><i>3 路径 · 循环控制</i>"]
        MOD["✎ module.md<br/><i>任务 · 执行模式 · 提炼配置</i>"]
    end

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% Layer 2 · 外部资源
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    subgraph L2["▌Layer 2 · 外部资源"]
        direction LR
        SKL[("⊞ MK Skill 库<br/>规范来源 · 只读")]
        REF[("⊞ 参考工程<br/>对比基准 · 只读")]
        TGT[("⊟ 目标工程<br/>代码写入")]
        MECH[("⊞ 机制源码<br/>提炼时指定 · 只读")]
    end

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% Layer 3 · Orchestrator
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    subgraph L3["▌Layer 3 · Orchestrator — 编排与决策"]
        ORC["🎯 Orchestrator<br/><i>读配置 · 派发子 Agent · 终止判断</i>"]
        MODE{{"模式判断<br/>module.md<br/>Skill提炼配置？"}}
        JUDGE{{"终止判断<br/>收敛 · 最大轮次 · diff 清零"}}
        WAIT(["⏸ 等待人工确认"])
        ORC --> MODE
        MODE -- "有内容<br/>Skill 提炼模式" --> PB
        MODE -- "为空<br/>代码生成模式" --> PA
        JUDGE -- "未收敛" --> WAIT -- "继续" --> ORC
    end

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% Layer 4 · 子 Agent 池
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    subgraph L4["▌Layer 4 · 子 Agent 池（.claude/agents/ 统一管理）"]
        direction LR
        subgraph PA["代码生成模式"]
            direction LR
            AG1["① Learn Agent<br/><i>识模块 · 读规范</i>"] --> AG2["② Generate Agent<br/><i>读 Skill · 写代码</i>"] --> AG3["③ Compare Agent<br/><i>git diff · 校验</i>"] --> AG4["④ Optimize Agent<br/><i>归档 Skill</i>"]
        end
        subgraph PB["Skill 提炼模式"]
            direction LR
            BG1["① Learn Agent<br/><i>机制源码解析</i>"] --> BG4["② Optimize Agent<br/><i>提炼 + 归档 Skill</i>"]
        end
        AG5["⑤ 发布脚本<br/><i>rm -rf + cp -r<br/>同步 product / skill_library</i>"]
    end

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% Layer 4.5 · 插件层
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    subgraph L45["▌Layer 4.5 · 插件层（.claude/plugins/ 通用能力）"]
        direction LR
        PL1["maven-analyzer<br/><i>Maven 解析</i>"]
        PL2["git-diff<br/><i>变更提取</i>"]
        PL3["version-manager<br/><i>版本管理</i>"]
        PL4["file-sync<br/><i>文件同步</i>"]
    end

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% Layer 5 · 文件交接层
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    subgraph L5["▌Layer 5 · 文件交接层（Agent 间通信媒介，按 {输出目录名} 隔离）"]
        direction LR
        W1["01_learn/{输出目录名}/<br/><i>learn_report.md · mechanism_analysis.md</i>"]
        W2["02_compare/reports/{输出目录名}/<br/><i>diff_report.md</i>"]
        W3["03_optimize/{输出目录名}/vX.Y.Z/<br/><i>SKILL.md · loop_state.md · update_log.md</i>"]
    end

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% Layer 6 · 产物层
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    subgraph L6["▌Layer 6 · 产物层"]
        direction LR
        P1["⭐ product/skill/{机制名}/<br/>标准 MK Skill · 可安装"]
        P2["📋 product/sample/{机制名}-vX.Y.Z/<br/>验证凭证 · 版本记录"]
    end

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% 生态回流
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    INSTALL[["◎ MK Skill 生态<br/>.trae/skills/MK开发流程/<br/>发布后在任意工程复用"]]

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% 数据流
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    L1 -- "配置 + 任务" --> ORC

    SKL -- "规范" --> AG1 & AG2 & BG1
    REF -- "参考" --> AG1 & AG3 & BG1
    MECH -- "机制 API" --> BG1
    TGT -- "已有代码" --> AG1
    AG2 -- "写入代码" --> TGT

    AG1 & BG1 -- "调用" --> PL1
    AG3 -- "调用" --> PL2
    AG4 & BG4 -- "调用" --> PL3
    AG5 -- "调用" --> PL4
    PL1 -. "Maven信息" .-> AG1 & BG1
    PL2 -. "diff报告" .-> AG3
    PL3 -. "版本目录" .-> AG4 & BG4

    AG1 & BG1 -- "learn_report.md" --> W1
    AG3 -- "diff_report.md" --> W2
    AG4 & BG4 -- "vX.Y.Z/" --> W3
    W1 & W2 & W3 -. "文件交接" .-> L4

    JUDGE -- "收敛" --> L6
    ORC -- "/publish" --> AG5
    AG5 --> P1
    P1 -. "先删后复制 · 安装" .-> INSTALL
    INSTALL -. "回流 · 下次生成复用" .-> SKL

    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    %% 样式
    %%━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
    classDef layer fill:#f0f4fa,stroke:#3b5bdb,stroke-width:1.5px
    classDef resource fill:#f8f9fa,stroke:#868e96,stroke-width:1px
    classDef orch fill:#fff9db,stroke:#f59f00,stroke-width:2px
    classDef agent fill:#ffffff,stroke:#4dabf7,stroke-width:1px
    classDef publish fill:#f1f3f5,stroke:#868e96,stroke-width:1px,stroke-dasharray:4
    classDef judge fill:#fff9db,stroke:#f59f00,stroke-width:1.5px
    classDef wait fill:#fff0f6,stroke:#e64980,stroke-width:1px
    classDef file fill:#f8f9fa,stroke:#adb5bd,stroke-width:1px,stroke-dasharray:4
    classDef product fill:#ebfbee,stroke:#2f9e44,stroke-width:1.5px
    classDef ecosystem fill:#f3f0ff,stroke:#7048e8,stroke-width:2px

    classDef plugin fill:#fff4e6,stroke:#e8590c,stroke-width:1px
    class PL1,PL2,PL3,PL4 plugin
    class L1,L2,L3,L4,L45,L5,L6 layer
    class SKL,REF,TGT,MECH resource
    class ORC orch
    class AG1,AG2,AG3,AG4,BG1,BG4 agent
    class AG5 publish
    class JUDGE judge
    class WAIT wait
    class W1,W2,W3 file
    class P1,P2 product
    class INSTALL ecosystem
```

---

## Agent 架构分层

```mermaid
block-beta
  columns 5

  L1["用户指令层"]:1
  C1["启动 CodePilot"]:1 C2["继续"]:1 C3["发布 Skill"]:1 space:1

  L2["编排层"]:1
  ORC["Orchestrator · 读配置 · 模式判断 · 派发 Agent · 终止判断"]:4

  L3["执行层"]:1
  block:PA[" "]:1
    columns 1
    PAH["Skill 提炼链路"]
    EA1["Learn Agent\n（机制源码解析）"]
    EA2["Optimize Agent\n（提炼归档 Skill）"]
  end
  block:PB[" "]:2
    columns 2
    PBH["── 代码生成链路 ──"]:2
    EB1["Learn Agent\n（识模块·读规范）"] EB2["Generate Agent\n（读Skill·写代码）"]
    EB3["Compare Agent\n（diff·校验偏差）"] EB4["Optimize Agent\n（修正Skill规则）"]
  end
  block:PC[" "]:1
    columns 1
    PCH["发布链路"]
    EC1["脚本命令\n（rm -rf + cp -r）"]
  end

  LP["插件层"]:1
  PL1["maven-analyzer（Maven解析）"]:1 PL2["git-diff（变更提取）"]:1 PL3["version-manager（版本管理）"]:1 PL4["file-sync（文件同步）"]:1

  L4["文件通信层"]:1
  F1["learn_report.md（模块信息）"]:1 F2["diff_report.md（偏差清单）"]:1 F3["vX.Y.Z / loop_state.md（Skill版本·迭代状态）"]:2

  L5["外部资源层"]:1
  R1["Skill库（只读）"]:1 R2["参考工程（只读）"]:1 R3["目标工程（读写）"]:1 R4["机制源码（只读）"]:1

  classDef compact font-size:11px
  classDef plugin fill:#fff4e6,stroke:#e8590c,stroke-width:1px
  class L1,C1,C2,C3,L2,ORC,LP,L4,F1,F2,F3,L5,R1,R2,R3,R4 compact
  class PL1,PL2,PL3,PL4 plugin
```

---

## 快速开始

### 前置条件

| 必需 | 说明 |
|------|------|
| MK Skill 库 | 克隆自 MK 平台 Skill 仓库 |
| 参考工程 | 一个完整的 MK 后端模块工程，作为对比基准和 Skill 提炼案例 |
| 目标工程 | 已有多模块脚手架（`-api` / `-core` / `-client`）的 MK Maven 工程 |
| 机制源码 | MK 平台机制模块源码，Skill 提炼时用于明确机制能力边界 |

> 目标工程无脚手架时，先发送：`请按照 MK后端模块脚手架 为 {模块名} 创建脚手架`

---

### 第一步：克隆本工程

```bash
git clone <本仓库地址>
cd CodePilot
```

---

### 第二步：配置 3 个路径

打开 `.claude/config.yml`，修改 **★ 新手配置区** 的 3 个路径：

```yaml
paths:
  target_project:    /你的路径/mk-km-xxx      # 生成代码写入此处
  reference_project: /你的路径/mk-km-review   # 对比基准 + Skill 提炼案例
  skill_library:     /你的路径/.trae/skills   # MK 规范来源
```

**只改这 3 行，其余不动。**

> 机制源码路径在 `00_workspace/input/module.md` 的 Skill 提炼配置里按任务填写，支持每次指定不同机制。

---

### 第三步：填写任务

打开 `00_workspace/input/module.md`，按目标选择模式：

**模式一 · 代码生成**（用已有 Skill 生成 Java 代码，每轮反馈优化 Skill）
```markdown
- [x] 增量项目

## 本次任务
为 KmNews 模块接入权限机制，补全 AuthRoles / AuthValidators 定义
```

**模式二 · Skill 提炼**（从机制源码提炼可复用规范，跳过代码生成）
```markdown
## Skill 提炼配置
- 分析子模块：
- 机制源码：/你的路径/mk-sys-auth
- 输出目录名：MK后端权限机制部署
```

> 模块名称、Key、包路径由 Agent 自动识别，无需手动填写。

---

### 第四步：启动

在 Claude Code 中发送：

```
启动 CodePilot
```

Agent 自动读取配置 → 识别任务模式 → 执行完整流程 → 每轮结束等待确认。

---

### 第五步：查看结果

```bash
cd /你的目标工程路径
git diff --stat   # 查看生成了哪些文件（代码生成模式）
```

Skill 产物在 `product/skill/`，确认质量后执行发布：

```
/publish
```

---

## 工程结构

所有中间产物均按 `{输出目录名}` 隔离，多机制并行时互不干扰。

```
CodePilot/
├── .claude/
│   ├── config.yml                    ★ 只改这里的 3 个路径
│   ├── instructions.md                 Orchestrator 执行指令（无需改动）
│   ├── commands/                       自定义交互指令集
│   │   └── publish.md                  /publish — 发布 Skill
│   ├── plugins/                        第三方能力插件（通用·可复用）
│   │   ├── maven-analyzer.md           Maven 多模块工程信息提取
│   │   ├── git-diff.md                 git diff 执行与结构化输出
│   │   ├── version-manager.md          vX.Y.Z 目录 + latest 软链接管理
│   │   └── file-sync.md                先删后复制文件同步
│   ├── agents/                         子 Agent 提示词（MK 业务逻辑）
│   │   ├── learn-agent.md              学习阶段：模块识别 + 规范摘要
│   │   ├── generate-agent.md           生成阶段：读 Skill · 写代码
│   │   ├── compare-agent.md            校验阶段：git diff · 对比报告
│   │   └── optimize-agent.md           优化阶段：Skill 归档 · 状态更新
│   └── skills/
│       ├── project_skill.md            Agent 自动维护的项目规范
│       ├── skill-gen-prompt.md         Skill 提炼三阶段提示词
│       └── references/                 命名约定与机制分类清单
│
├── 00_workspace/input/
│   └── module.md                     ★ 每次任务只填这里
│                                       生成代码直接写入 target_project（见 config.yml）
│
├── 01_learn/
│   └── {输出目录名}/                   学习报告 · 机制解析（每机制隔离）
│       ├── learn_report.md             代码生成模式：模块识别结果
│       └── mechanism_analysis.md       Skill 提炼模式：机制接入清单
│
├── 02_compare/reports/
│   └── {输出目录名}/                   diff 校验报告（每机制隔离）
│       └── diff_report.md
│
├── 03_optimize/
│   └── {输出目录名}/                   Skill 迭代版本（每机制隔离）
│       ├── loop_state.md               迭代状态（轮次、收敛判断）
│       ├── update_log.md               进化日志
│       ├── latest -> vX.Y.Z/           软链接，始终指向最新版本
│       ├── v1.0.0/
│       └── vX.Y.Z/
│
└── product/
    ├── skill/
    │   └── {机制名}/                 ★ 核心产物：标准 MK Skill（可安装）
    └── sample/
        └── {机制名}-vX.Y.Z/            验证凭证：版本记录
```

---

## 常见问题

**Q：Skill Loop 跑完一轮后怎么继续？**
回复「继续」，Agent 读取 `loop_state.md` 从上次断点接着跑，不会重头来过。

**Q：只想提炼 Skill，不生成代码？**
在 `module.md` 填写 `Skill 提炼配置` 区块，Agent 自动走提炼路径，跳过代码生成。

**Q：MK Skill 库更新了怎么办？**
```bash
cd /你的Skill库路径 && git pull
# 下次启动 CodePilot 自动读取最新规范
```

**Q：生成代码有问题怎么办？**
在 `module.md` 补充说明，重发「启动 CodePilot」，Agent 读取已有代码增量修正。

**Q：两种模式有什么区别？**
- **Skill 提炼模式**：`module.md` 填写 `Skill 提炼配置` 区块，从机制源码提炼规范，不生成业务代码，适合首次建立 Skill。
- **代码生成模式**：`module.md` 填写 `本次任务`，读取已有 Skill 生成 Java 代码，每轮 diff 对比后反馈优化 Skill，适合落地到具体工程。

**Q：如何发布 Skill 到 Skill 库？**
Skill 收敛后发送 `发布 Skill` 或直接输入 `/publish`，自动执行先删后复制同步至 `product/skill/` 和 `skill_library`。

---

## 外部资源说明

| 资源 | 配置位置 | 权限 | 使用阶段 |
|------|---------|------|---------|
| 目标工程 | `config.yml → paths.target_project` | 读写 | Generate：代码写入目标 |
| 参考工程 | `config.yml → paths.reference_project` | 只读 | Learn / Compare / Skill 提炼案例 |
| Skill 库 | `config.yml → paths.skill_library` | 只读 | Learn / Generate：规范来源 |
| 机制源码 | `module.md → Skill提炼配置 → 机制源码` | 只读 | Skill 提炼：按任务指定，可每次不同 |
