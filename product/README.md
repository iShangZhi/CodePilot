# product — 工程最终产物

本目录存放 CodePilot Hub 的两个产物：
- **后端 Skill**：核心产物，Skill 自进化的最终成果
- **验证代码记录**：验证凭证，证明 Skill 能产出符合规范的真实代码

---

## 产物一：后端 Skill（`skill/`）

**用途**：可直接安装到任意 MK 工程的标准 Skill，指导 Claude 生成同类后端代码。

**来源**：从 `03_optimize/latest/` 验证稳定后提升而来。

**格式**：与 `.trae/skills/MK开发流程/` 中现有 Skill 完全一致：

```
skill/
└── {模块名}后端部署/
    ├── SKILL.md                      # frontmatter(name+description) + 规范正文
    │                                 # 正文包含：实体概览、部署流程、验证清单、注释规范、占位符说明
    └── references/
        ├── ref-00-bundle-spec.md     # 全局包导入规范（所有 ref 均须遵守）
        ├── ref-01-{Entity}-spec.md   # 机制能力型：Entity→VO→API→Service→Controller
        ├── ref-02-{Entity}-integration.md  # 引擎集成型（如表单/流程引擎）
        └── ref-0N-{能力}-spec.md     # 无 Entity 的独立能力（枚举/工具类/配置/i18n）
```

**ref 文件内部结构**（每个代码单元）：
1. **核心要点**（说明 + 表格）
2. 代码块（含行内注释，说明"为什么"而非"做什么"）
3. **注意事项**列表

**安装方式**：
```bash
# 先删旧版本再复制，避免历史迭代中重命名的旧文件残留
rm -rf /你的skill库路径/MK开发流程/{模块名}后端部署 && \
cp -r product/skill/{模块名}后端部署 /你的skill库路径/MK开发流程/
```
> 安装完成后，下次在任意工程中启动 Claude Code，该 Skill 自动可用。

---

## 产物二：案例代码（`sample/`）

**用途**：记录每次 Skill Loop 生成的文件清单，作为 Skill 正确性的实物验证记录。

**注意**：完整代码在目标工程（`{target_project}`）中，此处只记录清单和说明。

```
sample/
└── {模块名}-vX.Y.Z/
    ├── README.md     # 模块信息、生成范围、对应 Skill 版本、完整代码位置
    └── files.md      # 新增/修改的文件清单（路径 + 关键类说明）
```
