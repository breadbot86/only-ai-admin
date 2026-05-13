# 思想记录：前端组件目录与组件说明界面

## 背景

AI 原生 admin 框架需要显式描述前端组件。组件不能只存在于代码、类型定义或零散文档中。

对于 admin 后台来说，表格、表单、搜索栏、功能栏、导入导出、文件上传、验证码、权限控件、租户切换、字典选择、状态标签等组件都承载了业务语义和后端关系。

因此，需要有一个统一位置介绍所有前端组件，让人类开发者和 AI Agent 都能理解：

- 有哪些组件
- 组件用于什么场景
- 组件有哪些属性
- 属性如何绑定数据、权限、租户、后端 contract
- 组件有哪些扩展点
- 组件有哪些限制和反模式
- 组件有哪些示例

## 结论

AI 原生 admin 框架应该同时具备两种组件说明形态：

1. 机器可读的 Component Registry / Component Catalog
2. 人类可浏览的组件说明界面 / Storybook-like Component Explorer

这两者不应割裂，而应来自同一个组件元数据源。

## 1. 机器可读 Component Registry

这是 AI Agent 的主要入口。

它可以是 JSON、YAML、数据库、代码生成产物或框架 introspection API。

它应该记录：

- 组件 ID
- 组件名称
- 组件类型
- 组件用途
- import / 使用入口
- props / attributes
- slots / children
- events / actions
- extension points
- data binding
- permission binding
- tenant binding
- backend contract binding
- validation rules
- audit binding
- examples
- anti-patterns
- related docs

示例：

```yaml
components:
  DataTable:
    type: data_display
    purpose: Display backend query result with pagination, sorting and row actions.
    props:
      dataSource:
        type: backend_query_contract
        required: true
      columns:
        type: table_column[]
        binds_to: model_fields
      rowActions:
        type: action[]
        requires:
          - permission_binding
          - audit_binding
    backend_bindings:
      - query_api
      - export_task
    extension_points:
      - toolbar_left
      - toolbar_right
      - row_action
      - batch_action
    docs:
      - docs/components/data-table.md
```

## 2. 人类可浏览组件说明界面

仅有 registry 对 AI 足够，但对人类开发者不够直观。

因此，admin 框架应提供一个组件说明界面，类似 Storybook / Design System Docs / Component Explorer。

这个界面应展示：

- 所有前端组件列表
- 组件分类
- 组件用途说明
- props / events / slots
- 权限和租户约束
- 后端 contract 绑定
- 扩展点
- 示例和反例
- 常见组合方式
- 可视化 demo
- 相关模块和页面使用情况

但这个界面不能只是视觉组件库文档。对于 admin 框架，它还必须展示组件的业务语义和后端依赖。

例如 DataTable 页面应展示：

- 可以绑定哪些后端 query contract
- 列如何来自 model field
- 搜索栏如何和 query 参数对应
- 行操作如何绑定 permission/action/audit
- 导出功能如何绑定 task center/file service

## 3. 单一元数据源

组件说明界面和 AI registry 不应该维护两套信息。

推荐做法：

```txt
Component Metadata Source
  ├── 生成 Component Registry，供 AI 读取
  ├── 生成 Component Explorer，供人浏览
  ├── 生成类型 / schema / 校验规则
  └── 生成组件文档
```

这样可以避免：

- 文档和代码不一致
- AI 读取的信息和人看到的信息不一致
- 组件属性更新后文档忘记更新
- 组件扩展点散落在不同地方

## 4. 组件目录不是可选项

对于 AI 原生 admin 框架，组件目录应该是一等能力。

原因：

1. AI 需要知道可用组件，而不是自己发明组件。
2. AI 需要知道组件属性和扩展点，而不是猜测 props。
3. AI 需要知道组件与后端 contract、权限、租户的绑定方式。
4. 人类需要有统一界面理解框架组件能力。
5. 组件使用情况可以反向帮助框架发现缺失能力和过时组件。

## 5. 建议命名

可以考虑以下概念：

- Component Registry：机器可读组件注册表
- Component Catalog：组件目录
- Component Explorer：人类可浏览组件说明界面
- Component Contract：组件契约
- Component Metadata：组件元数据

其中：

- Registry 偏 AI / 机器读取
- Explorer 偏人类浏览
- Contract 偏规则和约束
- Metadata 是共同源头

## 初步规则

1. AI 原生 admin 框架必须提供统一的前端组件目录。
2. 组件目录必须机器可读，供 AI Agent 查询和验证。
3. 框架应提供人类可浏览的组件说明界面，用于展示组件能力、示例、扩展点和后端绑定关系。
4. 组件说明界面和机器可读 registry 应来自同一个组件元数据源。
5. 组件说明不能只描述 UI props，还必须描述 data/permission/tenant/action/backend/audit 等语义绑定。
6. AI Agent 在生成前端页面前，应先查询组件目录，优先复用已有组件，而不是发明新组件。
