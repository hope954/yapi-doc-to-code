# Page Generation Task

## Goal

基于 `page.manifest.json` 为“用户列表”页面准备代码生成上下文。本阶段先消费 manifest 与 prompt，不要求直接输出完整页面代码。

## Inputs

- PRD：`docs/prd/user-list.pdf`
- Figma：`https://figma.com/design/example-user-list`
- API：`src/api/api.ts`
- Model：`src/models/model.ts`

## Manifest Summary

- 页面标题：用户列表
- 页面类型：`list`
- 模块：`user`
- 路由：`/user/list`
- UI 库：`Arco`

## Layout Plan

- 顶部是筛选区，字段包含 `keyword`、`status`
- 中间是表格区，字段包含 `userName`、`phone`、`status`、`createdAt`
- 表格上方保留工具栏，含“新建用户”按钮

## Data Binding Plan

- 列表数据来自 `getUserList`
- 状态切换动作来自 `updateUserStatus`
- 列表模型使用 `UserModel`
- 字段 `user_name` 映射到页面字段 `userName`

## Component Plan

- 使用 `PageContainer` 作为页面容器
- 使用 `SearchForm` 承载查询条件
- 使用 `UserTable` 承载列表与状态操作
- 优先复用项目现有组件与现有依赖

## TODOs And Conflicts

- TODO：确认“新建用户”按钮是否已有后端接口
- conflict：`api:user_name` 与 `figma:用户名` 已映射为 `userName`

## Output Requirement

- 输出目录建议为 `src/pages/user/list`
- 当前阶段目标文件仅为：
  - `page.manifest.json`
  - `cursor-page-prompt.md`
