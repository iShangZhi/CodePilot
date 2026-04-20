# product/skills — Skill 产物库

本目录存放每个机制提炼出的 SKILL，供 Generate Agent 读取以生成后端 Java 代码。

每个机制单独一个子目录，互不干扰，支持多机制并行迭代。

---

## 目录结构

```
product/skills/
├── README.md
│
├── MK后端权限机制部署/        # 机制子目录，名称与 module.md「输出目录名」一致
│   ├── SKILL.md               # 如何接入机制（Maven、基类、注解、配置）
│   └── references/            # 详细规范（SKILL.md 中 > 见 ref-XX 时按需读取）
│       ├── ref-00-bundle-spec.md
│       └── ...
│
├── MK后端权限机制服务清单/
│   ├── SKILL.md               # 接入后提供的 API 接口 / Controller 路由
│   └── references/
│
└── {下一个机制名}/
    ├── SKILL.md
    └── references/
```

---

## 子目录命名规则

子目录名 = `module.md` 中「Skill 提炼配置 → 输出目录名」字段的值。

Extract Agent 提炼完成后自动创建（若不存在）或覆盖写入（若已存在）。

---

## 写入规则

每次提炼（Extract）或修订（Patch）直接覆盖写入，无版本目录、无软链接。
SKILL.md 即当前权威版本，由 git 历史追溯变更记录。

---

## 与 Skill 库的关系

| 目录 | 定位 | 对外？ |
|------|------|--------|
| `product/skills/{机制名}/` | 本项目专用 SKILL，可人工编辑调整 | 是，Generate Agent 直接读取 |
| `{skill_library}/MK开发流程/{机制名}/` | MK 平台原始 Skill 库，只读 | 否，不可修改 |
