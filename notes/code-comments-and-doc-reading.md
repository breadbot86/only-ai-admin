# 思想记录：代码注释、模块文档沉淀与文档读取配置

## 背景

AI 原生框架不仅要显式化路由、组件、权限、前后端关系，也需要重新看待代码注释和文档沉淀。

传统项目中，代码注释和文档经常存在三个问题：

1. 注释随意，缺少结构
2. 文档写完后无人维护
3. AI Agent 不知道应该先读哪些文档，也不知道哪些文档与当前任务相关

如果 AI 原生框架希望支持长期演进，就必须把代码注释、模块文档和文档读取引导纳入框架规则。

## 代码注释不是解释语法，而是沉淀意图和边界

AI 原生框架不应该鼓励无意义注释，例如解释语言语法或重复代码表面含义。

不好的注释：

```txt
// loop through users
// call API
// set value
```

有价值的注释应该沉淀 AI 以后难以从代码表面推断的信息，例如：

- 为什么这样设计
- 为什么不能用另一种更简单的方式
- 这里依赖什么框架约定
- 这里是否涉及权限、租户、安全、审计
- 这里是否有历史兼容原因
- 这里是否是扩展点
- 这里是否是生成代码或禁止手改区域
- 修改这里会影响哪些模块或能力

核心原则：

> 注释不应解释代码正在做什么，而应解释代码为什么这样做、不能怎么改、改动会影响什么。

## 注释应该结构化

AI 原生框架可以定义一些标准注释标签，让 AI 更容易识别代码中的语义。

例如：

```txt
@ai-purpose
@ai-contract
@ai-invariant
@ai-extension-point
@ai-generated
@ai-editable
@ai-do-not-edit
@ai-permission
@ai-tenant-boundary
@ai-audit
@ai-doc
@ai-decision
@ai-pitfall
```

这些标签不一定绑定具体语言，可以被不同语言实现映射到对应注释风格。

例如：

```txt
@ai-invariant: exported data must be scoped by tenant_id.
@ai-permission: requires user.export permission.
@ai-doc: docs/modules/user/export.md
@ai-pitfall: do not reuse list query directly; export requires stronger data-range validation.
```

这样 AI Agent 在修改代码时能快速识别关键边界。

## 注释需要和文档互相引用

注释不应该承载完整文档，否则会让代码臃肿。

更合理的方式是：

- 代码注释记录局部关键语义
- 模块文档记录完整背景、设计、流程和规则
- 注释通过 `@ai-doc` 指向对应文档
- 文档反向记录相关代码入口

例如：

```txt
@ai-doc: docs/modules/billing/import-export.md
@ai-decision: ADR-0012
```

这样 AI 可以从代码跳转到完整上下文。

## 每个模块开发后必须沉淀文档

AI 原生框架应该规定：模块不是只有代码目录，还必须有模块文档。

每个模块至少应该沉淀：

- 模块目的
- 模块边界
- 不负责什么
- 核心能力
- 对外接口
- 前端入口
- 后端 contract
- 权限与角色
- 租户边界
- 数据模型
- 扩展点
- 公共能力依赖
- 关键流程
- 错误处理
- 测试/验收场景
- 常见坑
- 相关决策记录

这可以形成模块级长期记忆，避免 AI 每次都从代码重新推断。

## 文档读取引导必须配置化

AI Agent 最大的问题不是“能不能读文档”，而是“不知道该读哪些文档”。

因此，AI 原生框架应该维护文档读取配置，告诉 AI 在不同任务下应该读取哪些文档。

例如：

```yaml
doc_reading:
  default:
    - README.md
    - standards/ai-native-framework-rules-v0.1.md

  when_creating_module:
    - docs/framework/module-contract.md
    - docs/framework/documentation-lifecycle.md
    - docs/framework/permission-model.md

  when_modifying_frontend:
    - docs/framework/frontend-backend-coupling.md
    - docs/framework/component-contract.md
    - docs/framework/permission-model.md

  when_adding_import_export:
    - docs/common-capabilities/file-management.md
    - docs/common-capabilities/import-export.md
    - docs/framework/task-center.md
    - docs/framework/audit.md
```

这类配置可以存在于：

```txt
.ai-docs.yaml
.agent-docs.yaml
only-ai-admin.docs.yaml
```

具体名字以后可以再定。

核心是：

> 文档不只是存在，还必须能被 AI 正确发现、正确排序、按任务加载。

## 文档索引应成为框架产物

框架应该维护一个文档索引，记录：

- 文档 ID
- 文档类型
- 面向读者：human / ai / both
- 关联模块
- 关联功能
- 关联接口
- 关联权限
- 关联代码入口
- 适用任务场景
- 最近验证时间
- 是否过期

示例：

```yaml
documents:
  - id: user_import_export
    path: docs/modules/user/import-export.md
    type: module_capability
    audience: [human, ai]
    module: user
    related_capabilities:
      - file_management
      - import_export
      - task_center
    related_permissions:
      - user.import
      - user.export
    read_when:
      - modifying_user_import
      - modifying_user_export
      - changing_file_service
```

## 初步规则

1. AI 原生框架应定义代码注释规则，注释重点沉淀意图、约束、边界、风险和文档链接。
2. 注释应尽量结构化，让 AI Agent 能稳定识别关键语义。
3. 生成代码、可编辑区域、禁止编辑区域、权限边界、租户边界、扩展点等应有明确注释或元数据标识。
4. 每个模块开发后必须沉淀模块文档，记录模块目的、边界、接口、权限、租户、扩展点、流程、测试和常见坑。
5. 文档读取引导必须配置化，让 AI 能根据任务类型知道应该读取哪些文档。
6. 框架应维护文档索引，记录文档与模块、功能、接口、权限、代码入口之间的关系。
7. 文档是否缺失、过期或未被配置到读取路径中，都应该进入框架验证范围。
