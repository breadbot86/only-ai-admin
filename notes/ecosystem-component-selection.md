# 思想记录：生态组件选型与组合

## 背景

AI 原生 admin 框架不应该把所有能力都从 0 开发。

在很多领域，例如 ORM、权限、认证、表单、表格、图表、任务队列、缓存、日志、监控、测试、文档生成等，生态中已经存在大量优秀组件。

框架真正重要的能力不是重复造轮子，而是：

1. 能识别优秀组件
2. 能判断组件是否适合当前框架目标
3. 能判断组件之间是否兼容
4. 能把多个组件组合成更高价值的整体
5. 能把组件能力、约束和使用方式显式暴露给 AI Agent

## 核心观点

AI 原生框架的组件策略应该是：

> 优先复用成熟生态组件，但必须把选型、版本、兼容关系、组合价值和使用约束显式化。

不是所有优秀组件都适合进入框架。

一个组件是否值得采用，不只看它单点能力强不强，还要看它和框架其他组件组合后，是否形成 1 + 1 > 2 的系统能力。

## 组件选型应关注的问题

### 1. 成熟度

- 是否被广泛使用
- 是否有稳定维护者
- 是否有足够 issue / PR 活跃度
- 是否有清晰文档
- 是否有长期维护迹象

### 2. 版本兼容性

- 是否支持目标运行时版本
- 是否支持目标语言版本
- 是否支持目标框架版本
- 是否与其他关键依赖冲突
- 是否有明确升级路径

### 3. 契约清晰度

- API 是否稳定
- 输入输出是否明确
- 错误是否可预测
- 是否有类型、schema 或文档
- 是否适合被 AI Agent 理解和调用

### 4. 可组合性

- 是否能与其他核心组件自然协作
- 是否容易形成数据流闭环
- 是否能共享 schema、类型、元数据或约定
- 是否减少胶水代码
- 是否降低上下文负担

### 5. 可替换性

- 是否会造成严重 vendor lock-in
- 是否可以抽象出 adapter
- 是否可以在未来替换
- 是否可以与其他实现共存

### 6. 可观测性与可调试性

- 是否容易定位错误
- 是否有结构化错误
- 是否方便日志、trace、metrics
- 是否方便 AI Agent 解释和修复问题

### 7. 安全性和边界

- 是否有已知安全风险
- 是否默认安全
- 是否容易误用
- 是否能显式表达权限、数据边界和副作用

## 组件组合不是堆叠

AI 原生框架不能只是把一堆流行组件拼在一起。

真正有价值的组合应该满足：

- 组件之间有清晰边界
- 组件之间有明确数据契约
- 组件之间能共享上下文
- 组件之间的组合方式能被框架解释
- 组件之间的版本关系能被框架验证
- 组件之间的最佳实践能被沉淀成规则

例如 ORM 不只是数据库访问工具。它可能影响：

- 数据模型定义
- API 生成
- 表单生成
- 表格列生成
- 权限判断
- 审计日志
- 数据校验
- 测试数据生成
- 文档生成

如果 ORM 的 schema 可以和 admin UI、权限系统、API contract、测试数据生成共享，那么它就不只是一个独立组件，而是整个框架语义层的一部分。

这就是 1 + 1 > 2。

## AI 原生框架应维护组件关系图

框架应该显式记录组件之间的关系，例如：

```yaml
components:
  orm:
    selected: example-orm
    version: "^1.8"
    provides:
      - data_model_schema
      - query_api
      - migration_api
    consumed_by:
      - admin_table_generator
      - form_generator
      - permission_engine
      - api_contract_generator
    constraints:
      - schema must be introspectable
      - migration state must be queryable
      - query errors must be mappable to structured framework errors
```

这样 AI Agent 能知道：

- 为什么选这个组件
- 它提供什么能力
- 谁依赖它
- 它的变更会影响谁
- 它和其他组件如何协作

## 组件选型也应该有决策记录

每个关键组件的采用都应该有 ADR / Decision Log：

```yaml
decision:
  title: Choose ORM as data model source of truth
  context:
    - Admin framework needs data model metadata.
    - Form/table/API generation all depend on model schema.
  options:
    - Build custom model layer
    - Reuse existing ORM
    - Support multiple ORM adapters
  decision:
    - Reuse mature ORM through adapter interface.
  consequences:
    - Framework gains migration/query ecosystem.
    - Need adapter contract to avoid lock-in.
    - ORM schema becomes part of framework semantic layer.
```

这可以避免未来 AI Agent 或开发者忘记为什么做这个选择。

## 初步规则

1. AI 原生框架应优先复用成熟生态组件，而不是默认从 0 开发。
2. 组件选型必须显式记录原因、版本、约束、替代方案和后果。
3. 组件之间的依赖关系必须可查询、可解释、可验证。
4. 框架应关注组件组合后的系统能力，而不是单个组件的孤立能力。
5. 关键组件必须通过 adapter / contract 接入，避免框架被单一实现锁死。
6. 被选用组件必须能为 AI Agent 暴露足够清晰的能力、约束、错误和文档。
7. 组件升级、替换、废弃必须进入框架生命周期管理。
