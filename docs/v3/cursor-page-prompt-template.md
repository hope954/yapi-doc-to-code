# `cursor-page-prompt.md` V1 模板

用于把 `page.manifest.json` 转成可直接交给 Cursor 的页面实现 prompt。V3.0 先定义 prompt，不要求直接生成完整页面代码。

## 固定章节

```md
# Page Generation Task

## Goal

## Inputs

## Manifest Summary

## Layout Plan

## Data Binding Plan

## Component Plan

## TODOs And Conflicts

## Output Requirement
```

## 章节映射

- `Goal`：来自 `page.title`、`page.pageType`、`generation.status`
- `Inputs`：来自 `sourceContext`
- `Manifest Summary`：来自 `page`、`inference`
- `Layout Plan`：来自 `layout`
- `Data Binding Plan`：来自 `dataBinding`
- `Component Plan`：来自 `componentPlan`
- `TODOs And Conflicts`：来自 `todos`、`conflicts`
- `Output Requirement`：来自 `generation.targetFiles`、`generation.outputDir`

## 编写要求

- 明确页面类型、模块、路由、UI 库。
- 明确应优先复用项目现有组件和现有依赖。
- 明确哪些字段、动作、布局来自哪个源。
- 明确 TODO 和 conflict，禁止静默忽略。

## Cursor 使用方式

1. 先读取 `page.manifest.json`。
2. 按本模板生成 `cursor-page-prompt.md`。
3. 后续页面代码生成应以该 prompt 为直接输入。
