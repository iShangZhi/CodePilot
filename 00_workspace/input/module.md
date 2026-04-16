# 本次任务

> 每次只填这里，发送「启动 CodePilot」即可启动。
> 模块名称、Key、包路径由 Agent 自动识别，无需手动填写。

---

## 项目类型（只选一个）

- [ ] **全新项目** — 目标工程只有脚手架，全量生成所有文件
- [x] **增量项目** — 已有代码，仅新增 / 修改指定部分

---

## 本次任务

帮我提炼下面的任务
角色常量(统一)
	XX AuthConstant
角色定义
@AuthRoles
	{ROLE}_ADMIN
管理员，继承所有角色
	{ROLE}_SETTING
后台管理员，获得后台菜单入口
	{ROLE}_DEFAULT
所有已登录用户均拥有
权限校验器
@AuthValidators
	@AuthValidator
	@ValidateParam
自定义拦截器 
AbstractValidateProcess
	AbstractValidateProcess
	
---

## Skill 提炼配置（填写此区块则自动切换为 Skill 提炼模式，跳过代码生成）

- 分析子模块：
- 机制源码：/Users/ibyte/workspace/MK-PaaS/mk-sys-auth
- 输出目录名：MK后端权限机制部署

---

## 补充说明

（特殊约束、已知问题等；没有可留空）
