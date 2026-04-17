# workspace — 任务输入区

本目录是每次启动 CodePilot 时的**任务输入**，Agent 从 `input/module.md` 读取本次任务目标。

> 代码规范来自 MK 后端 Skill 库（路径见 `config.yml`）
> 代码结构对比基准来自参考工程（路径见 `config.yml`）
> 机制接入情况由 Agent 从目标工程源码中自动识别

---

## 使用方式

1. 填写 `input/module.md`（模块基本信息 + 本次任务描述）
2. 发送「启动 CodePilot」，Agent 自动执行四步闭环

## 目录结构

```
workspace/
└── input/
    └── module.md   # 每次任务前填写（模块信息 + 任务描述）
```
