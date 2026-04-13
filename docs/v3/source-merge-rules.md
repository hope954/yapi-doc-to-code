# 三源融合规则

V3 以三源合并为基础：
- API：数据真实性来源
- PRD：业务语义来源
- Figma：布局与视觉来源

## 基本原则

- 能落地到 API 的字段和动作，优先按 API 接入。
- PRD 决定字段含义、业务规则、状态语义。
- Figma 决定区域结构、控件类型、视觉层次。

## 字段融合

- PRD 与 API 均存在：直接绑定，并保留 PRD 语义说明。
- PRD 有字段但 API 无：保留字段，写入 `TODO(api-missing-field)`。
- API 有字段但 PRD 无说明：接入字段，写入 `TODO(prd-missing-semantic)`。
- Figma 显示字段但 PRD/API 都无：保留布局占位，写入 `TODO(unconfirmed-field)`。

## 动作融合

- Figma 有按钮且 API 有对应动作：建立 `actionMappings`。
- Figma 有按钮但 API 无动作：保留结构，写入 `TODO(api-missing-action)`。
- API 有动作但 Figma 未体现：允许进入 `componentPlan`，并写入 `TODO(figma-missing-action)`。

## 命名冲突

- 字段名不一致但语义一致：允许映射，记录到 `fieldMappings`。
- 无法确认是否同义：写入 `conflicts`，不要自动合并。
- 冲突项建议记录：`field`、`sources`、`decision`、`note`。

## 输出要求

- 缺失项优先转成 `todos`。
- 多源不一致优先转成 `conflicts`。
- 不因为局部缺失而放弃整个 manifest。
