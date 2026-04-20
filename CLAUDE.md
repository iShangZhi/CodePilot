# CodePilot Hub

CodePilot 是 Skill 自进化驱动的 Java 后端代码生成平台。
**范围限定：只处理后端 Java 代码，不生成任何前端代码。**

---

## 触发词

「启动 CodePilot」— 读取 `.claude/config.yml` 和 `workspace/input/module.md`，然后加载并执行 `.claude/skills/codepilot-workflow.md` 中的完整主流程。

---

## 核心约束

1. **路径来源**：所有外部路径从 `.claude/config.yml` 的 `paths` 区块读取，不硬编码
2. **只读外部**：不修改 `reference_project` 和 `skill_library` 下任何文件
3. **SKILL 即权威**：Generate Agent 不读目标工程源码，只读 SKILL，全量覆盖生成
4. **每阶段停止**：每条主线完成后停止，等待人工确认，不自动进入下一步

---

## Agent 入口

| Agent | 提示词文件 |
|-------|-----------|
| Extract Agent | `.claude/agents/extract-agent.md` |
| Generate Agent | `.claude/agents/generate-agent.md` |
| Patch Agent | `.claude/agents/patch-agent.md` |
