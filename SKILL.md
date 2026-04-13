---
name: yapi-doc-to-code
description: >-
  V1 负责拉取鉴权后的 YApi 文档并生成业务 `src/api/api.ts` 与
  `src/models/model.ts`；V3 在此基础上整合 PRD、Figma、api.ts、model.ts，
  输出 `page.manifest.json` 与 `cursor-page-prompt.md`，用于页面生成前的统一上下文准备。
---

# YApi 文档生成与 V3 页面上下文 Skill

本 skill 保留 V1 能力，并补充 V3 能力：
- V1：从鉴权后的 YApi 文档生成业务 `api.ts` / `model.ts`
- V3：基于 PRD、Figma、V1 产物整理页面上下文，输出 `page.manifest.json` 与 `cursor-page-prompt.md`

## V1 能力：YApi 文档驱动生成 `api.ts` / `model.ts`

面向聊天驱动：用户在对话中提供 **YApi 项目 id（`_yapi_uid`）** 与 **token（`_yapi_token`）**（及必要时 **YApi 服务根地址 base URL**），代理仅将这些信息用于拉取 YApi 在线接口文档；据此生成业务后端接口对应的 **`src/api/api.ts`** 与 **`src/models/model.ts`**。生成的是业务请求层，不是调用 YApi 开放接口本身的客户端代码。本能力为 Skill，不要实现为 Rule。

## 运行前提

- 在线拉取时，运行环境须具备可用网络访问。
- 无法联网时，向用户说明限制，并尽量退化为基于其提供的导出内容生成。
- YApi 鉴权信息仅用于拉取文档，不得写入生成代码。

## YApi 鉴权优先级

- 当用户提供 `_yapi_uid` 与 `_yapi_token` 时，优先使用 Cookie 鉴权：`Cookie: _yapi_token=...; _yapi_uid=...`。
- 若用户明确指定其他鉴权方式，按用户要求执行；否则保持 Cookie 优先。

## 鉴权自动探测流程

1. 先用轻量接口做可用性探测。
2. 若返回登录态错误，自动切换并重试 Cookie 鉴权。
3. 一旦某种方式成功，本次会话固定使用该方式继续拉取。
4. 若所有方式失败，向用户说明失败原因与已尝试方案，再退化为让用户提供可访问的导出文档内容。

## V1 范围

支持：
- 仅 YApi 开放平台/同源接口拉取与业务 `api.ts` / `model.ts` 生成。
- 用户仅在聊天中提供 `_yapi_uid`、`_yapi_token`；可选提供 YApi 根 URL。
- 输出路径固定：`src/api/api.ts`、`src/models/model.ts`。

不支持：
- 非 YApi 平台直接替代 V1 拉取流程。
- 把本流程写成 Rule。

## 规范优先级

1. 若用户在本次对话中提供了 Markdown 代码生成规范，则优先按其执行。
2. 否则读取并遵循本 skill 内置规范：`references/default-codegen-spec.md`。

## V1 核心工作流

1. 收集上下文：读取 `_yapi_uid`、`_yapi_token`，缺失时简短追问一次；必要时补 `base URL`。
2. 拉取文档：按鉴权规则调用 YApi 接口，获取项目、接口列表与必要详情。
3. 整理结构：抽取 method、path、标题、说明、path/query/body/form/header 参数、响应结构线索。
4. 生成代码：按规范生成业务 `src/api/api.ts` 与 `src/models/model.ts`。
5. 写入文件：默认覆盖该职责范围内的导出；用户另有说明时按用户要求处理。

## V1 生成物要点

详细规则见 `references/default-codegen-spec.md`。

- `api.ts`：TypeScript 业务请求层；区分 path / query / body / form-data / headers；生成请求与响应类型；尽量少 `any`。
- `model.ts`：`ModelBase`、`DataModel`、`Column` 从 `@model-base/core` 导入；按既有规范生成模型与字段映射。

## V1 降级与容错

- 文档字段缺失时不中断整次生成。
- 用 `optional`、`unknown` 或带 `TODO:` 的注释标明不确定处。
- 仍输出尽可能可编译、可调用的代码。

## V3 能力：页面生成上下文准备

V3 不直接生成完整页面代码，只负责整合输入、推断页面信息并产出中间物。
V3 不是替代 V1，而是建立在 V1 产物之上的补充能力。
V3 的关键输入之一是 V1 已生成的 `api.ts` / `model.ts`。

### V3 输入

必需输入：
- PRD（PDF 或飞书在线文档）
- Figma 页面链接
- V1 已生成的 `api.ts`
- V1 已生成的 `model.ts`

默认优先读取 V1 默认输出路径：
- `src/api/api.ts`
- `src/models/model.ts`

用户显式指定时可覆盖。

可选覆盖项：
- `outputDir`
- `referencePagePath`
- `uiLibrary`
- `pageType`

### 输入策略

- 默认优先自动推断，尽量减少手填参数。
- 只有推断不稳或信息缺失时，才要求补充覆盖项。
- 覆盖项一旦提供，以用户输入为准。

### 三源职责

- API：数据真实性来源。
- PRD：业务语义来源。
- Figma：布局与视觉来源。

### 推断规则

- `pageType`：从 PRD / Figma / API 联合推断。
- `module`：从 Figma 页面名 / PRD 标题 / API 命名推断。
- `uiLibrary` 优先级：
  1. 用户指定
  2. 项目已有依赖
  3. 默认：Arco > Element > Ant

### 输出物

V3.0 目标是先产出统一上下文文件，本阶段不要求直接生成完整页面代码。
- `page.manifest.json`：统一页面中间层，承载页面类型、模块、字段、动作、数据来源、映射、冲突与 TODO。
- `cursor-page-prompt.md`：给 Cursor 生成页面代码的 prompt，基于 manifest 组织实现指令。

### 冲突处理

- PRD 有字段但 API 无：标记 `TODO(api-missing-field)`。
- Figma 有按钮但 API 无动作：保留结构并标记 `TODO(api-missing-action)`。
- API 有字段但 PRD 无说明：接入并标记 `TODO(prd-missing-semantic)`。
- 命名不一致：允许映射，并记录 `conflict` / `TODO`。

### 失败与兜底

- 缺少 PRD / Figma / `api.ts` / `model.ts` 时要明确处理缺失项。
- 推断不稳时不能静默高风险猜测。
- 优先输出带 `TODO` / `conflict` 的结果。
- 关键输入缺失且无法替代时可中断，并给出最小补充清单。

## 自动 / 手动触发

- V1：当用户要根据接口文档生成 `api.ts` / `model.ts` 时触发。
- V3：当用户已具备 PRD、Figma 与 V1 产物，希望生成页面上下文或 `cursor-page-prompt.md` 时触发。
- 自动：当用户提到 YApi、api.ts、model.ts、请求层、鉴权文档、PRD、Figma、页面生成、manifest、页面 prompt 等相关意图时，应优先匹配本 skill。
- 手动：用户可输入 `yapi-doc-to-code` 或本 skill 名称显式触发。

## 附加资源

- 默认代码生成细则：`references/default-codegen-spec.md`
