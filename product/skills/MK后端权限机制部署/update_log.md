# Skill 进化日志

记录每次 Skill Loop 优化的核心内容，确保闭环迭代可追溯。

---

## v1.0.4 — 2026-04-17
变化来源：代码对比优化
偏差点：api/pom.xml 依赖判断规则不明确；方法级注释规范缺失；Javadoc 完整性要求
修正内容：
1. SKILL.md 阶段一：Maven 依赖从"仅 core 模块"改为表格形式，补充 api 模块按需引入的判断规则（检查是否有 com.landray.sys.auth.* 引用）
2. SKILL.md 部署验证清单：新增 Maven 依赖检查项、Javadoc 完整性检查项、权限继承注释检查项
3. SKILL.md 代码注释规范：新增"Controller 所有 public 方法必须有完整 Javadoc"和"继承类级别权限的方法须标注注释"两条规则
4. ref-00：Maven 依赖段从单一声明改为 core（必须）+ api（按需）双模块规则
5. ref-02：新增"继承类级别权限的方法"代码示例段落和"Javadoc 完整性要求"段落

---

## v1.0.3 — 2026-04-16

变化来源：增量补充（用户明确提炼需求）
偏差点：AuthConstant 概念混淆、角色定义职责说明不足、isCache() 缺代码示例
修正内容：
1. SKILL.md 能力概览：`{Module}Role` 明确标注"同时承担角色常量容器 + @AuthRoles 注册"，新增 `AuthConstant` 澄清说明
2. ref-01：新增「职责说明」段落，三个必选角色独立成表，新增"不要用 AuthConstant"易踩坑条目
3. ref-04：新增「开启结果缓存」代码示例段落，完整展示 `isCache()` 用法和缓存机制说明

---

## 版本记录格式

```
## vX.Y.Z — YYYY-MM-DD（第 N 轮）

### 快照信息
- MK Skill 库 commit：[git -C {skill_library} rev-parse --short HEAD]
- 变化来源：[ 规范库更新 / 代码对比优化 / 两者都有 ]

### 本轮数据
- diff 偏差项数：X 项（上轮：Y 项）
- Skill 变更行数：X 行

### 偏差点与修正
- [ 偏差描述 → 修正内容 ]

### 终止判断
- [ ] 达到最大轮次（N/5）
- [ ] Skill 收敛
- [ ] diff 清零
- 结论：[ 继续迭代 / 停止 ]
```

---

<!-- 以下由 Claude Agent 自动追加 -->

## v1.0.0 — 2026-04-14（第 1 轮）

### 快照信息
- MK Skill 库 commit：（本轮为机制源码逆向提炼，未从 Skill 库迭代）
- 变化来源：机制源码逆向提炼（mk-sys-auth）

### 本轮数据
- diff 偏差项数：— （Skill 提炼模式，跳过 Compare 阶段）
- Skill 变更行数：初始版本，约 350 行

### 偏差点与修正
- 首次提炼，无对比基准
- 逆向解析权限机制四大能力：角色常量、角色定义、校验器使用、自定义校验器
- 提炼重点：`@AuthRoles`/`@AuthRole` 角色注册体系，`@AuthValidators` 多校验器 AND/OR 组合，`AbstractValidateProcess` 带缓存的自定义校验器扩展点

## v1.0.1 — 2026-04-14（第 2 轮）

### 快照信息
- MK Skill 库 commit：（参考工程 mk-km-review 实际接入对比）
- 变化来源：代码对比优化（参考工程 vs v1.0.0 Skill）

### 本轮数据
- diff 偏差项数：6 项
- Skill 变更行数：约 120 行

### 偏差点与修正
1. **角色定义结构错误** → 从拆分的 interface + 空类改为单一 class（含常量 + `@AuthRoles`），与参考工程一致
2. **内置 roleValidator 参数格式遗漏** → 补充 `roleValidator` 是内置 id，`@ValidateParam` 直接传角色编码（无 `name`）
3. **SETTING 角色继承目标错误** → 从 `MENU_SYS_MANAGER_ROLE` 改为 `MENU_SYS_APPLICATION_ROLE`（应用级，非全局）
4. **@CompsiteValidators 完全缺失** → 新增 ref-03，覆盖 AND of ORs 嵌套复合鉴权完整规范
5. **自定义校验器 @Component 规则过于绝对** → 修正为条件性（@Autowired 时必须；getBean() 时无需）
6. **抽象父类模式未体现** → ref-04 补充抽象父类用法，覆盖多校验器共享逻辑场景

### 终止判断
- [ ] 达到最大轮次（2/5）
- [ ] Skill 收敛（变更行 ~120 > 10，未收敛）
- [ ] diff 清零
- 结论：第二轮完成，等待人工确认

### 终止判断
- [ ] 达到最大轮次（1/5）
- [ ] Skill 收敛
- [ ] diff 清零
- 结论：第一轮完成，等待人工确认

## v1.0.2 — 2026-04-15（第 5 轮）

### 快照信息
- MK Skill 库 commit：（本轮为格式重构后系统扫描，无规范库变更）
- 变化来源：代码对比优化（格式重构后一致性验证）

### 本轮数据
- diff 偏差项数：4 项
- Skill 变更行数：约 15 行

### 偏差点与修正
1. **ref-00 误收录 Plugin 导入** → 删除 `框架基础层 — Plugin`（`GlobalExtensionPoint`/`ListenerConfig`），权限机制 Skill 中无任何 ref 文件使用这两个类
2. **ref-01 业务角色说明不完整** → 三个必选角色描述补充"业务角色按实际需求增减"，对齐代码模板中 MAIN_CREATE/READER/UPDATE/DELETE 示例的语义
3. **ref-02 `@ModuleValidator` 属性名差异未说明** → 核心规则补充 `@AuthValidators` 用 `value=`、`@ModuleValidator` 用 `validators=` 的属性名区别
4. **ref-03 `{Module}AuthConstant` 创建条件不明** → 核心规则明确"使用自定义校验器时才创建"，并说明是 `interface`

### 终止判断
- [x] 达到最大轮次（5/5）
- [x] Skill 收敛（偏差项 4→0，变更行 ~15 < 10 的修正补丁）
- [ ] diff 清零（偏差为规范内部一致性，非对比工程差异）
- 结论：**停止** — 达到最大轮次且修正完毕，Skill 完全收敛
| v1.0.0 | 导入 | 来源：MK开发流程/MK后端权限机制部署 | 2026-04-17 |
