# `page.manifest.json` V1

用于承接 V3.0 的统一页面上下文。它是页面生成前的中间层，不是最终页面代码。

## 顶层结构

```json
{
  "page": {},
  "sourceContext": {},
  "inference": {},
  "layout": {},
  "dataBinding": {},
  "componentPlan": {},
  "generation": {},
  "todos": [],
  "conflicts": []
}
```

## 字段定义

### `page`

- 页面基础信息。
- 建议包含：`id`、`name`、`route`、`module`、`pageType`、`title`。

### `sourceContext`

- 输入来源与定位信息。
- 建议包含：`prd`、`figma`、`api`、`model`。
- 每项建议记录：`sourceType`、`pathOrUrl`、`summary`。

### `inference`

- 推断结果与置信度。
- 建议包含：`pageType`、`module`、`uiLibrary`、`route`、`pageName`。
- 每项建议记录：`value`、`confidence`、`evidence`。

### `layout`

- 页面结构与区域规划。
- 建议包含：`sections`。
- `sections` 可描述 `search`、`table`、`detail`、`form`、`toolbar`、`sidebar` 等区域。

### `dataBinding`

- 视图与接口/模型的绑定关系。
- 建议包含：`apis`、`models`、`fieldMappings`、`actionMappings`。

### `componentPlan`

- 组件级实现计划，不含最终代码。
- 建议包含：`uiLibrary`、`containers`、`components`。
- 每个组件建议记录：`name`、`role`、`propsFrom`、`notes`。

### `generation`

- 代码生成约束与目标输出。
- 建议包含：`outputDir`、`referencePagePath`、`targetFiles`、`status`。
- `targetFiles` V3.0 默认应包含：
  - `page.manifest.json`
  - `cursor-page-prompt.md`

### `todos`

- 待确认或待补全事项列表。
- 每项建议包含：`id`、`title`、`source`、`severity`、`suggestion`。

### `conflicts`

- 多源冲突记录。
- 每项建议包含：`id`、`field`、`sources`、`decision`、`note`。

## 最小要求

- 顶层九个字段都应存在。
- 信息不完整时允许空对象、空数组或 `TODO`。
- 不确定信息不要硬猜，优先落到 `todos` 或 `conflicts`。
