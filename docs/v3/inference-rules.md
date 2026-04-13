# V3 推断规则

## `pageType`

联合 PRD、Figma、API 推断：
- 出现列表、筛选、分页、批量操作，优先判定为 `list`
- 出现新增/编辑表单，优先判定为 `form`
- 出现单条信息展示、分组信息块，优先判定为 `detail`
- 同时存在列表和表单弹窗，可判定为 `crud`

低置信度时：
- 不静默硬猜
- 写入 `inference.pageType`
- 同时写入 `todos`

## `module`

联合以下信息推断：
- Figma 页面名
- PRD 标题
- API 命名空间、Tag、路径前缀、函数名

优先使用业务名词，不使用技术名词。

## `uiLibrary`

优先级：
1. 用户指定
2. 项目已有依赖
3. 默认：Arco > Element > Ant

若项目同时存在多个 UI 库：
- 优先选当前参考页面使用的库
- 若仍不稳，写入 `todos`

## `route`

推断来源：
- PRD 菜单名称
- Figma 页面层级
- 参考页面路径
- 模块名

默认规则：
- 使用 kebab-case
- 后台页面建议以 `/<module>` 或 `/<module>/<page-name>` 表达

## `pageName`

推断来源：
- 页面标题
- 模块名
- 页面类型

默认规则：
- 使用 PascalCase
- 示例：`UserListPage`、`OrderDetailPage`

## 低置信度处理

- `confidence < 0.7` 视为低置信度。
- 低置信度项必须记录 `evidence`。
- 低置信度项不得直接作为唯一结论，需同时输出 `TODO` 或待确认说明。
