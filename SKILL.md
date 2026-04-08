---
name: yapi-doc-to-code
description: >-
  Fetches authenticated YApi project documentation using project id and token from
  the chat, then generates TypeScript src/api/api.ts (business backend axios request
  layer with param/response types and functions, not YApi open-API client code) and
  src/models/model.ts (ModelBase/DataModel/Column from @model-base/core). Use when
  the user mentions YApi, YApi 拉取, YApi 鉴权文档,
  yapi-fetch, 生成 api, 自动生成 api, 生成 api.ts, 生成 model.ts, 请求层生成,
  根据接口文档生成代码/模型/请求层, 根据鉴权后的接口文档生成代码, ModelBase,
  operationId, or wants API/model codegen from online YApi docs without editing
  project config files. First version: YApi only; no Vue/pages; not a Rule. Manual
  trigger: skill name yapi-doc-to-code.
---

# YApi 文档驱动生成 api.ts / model.ts

面向聊天驱动：用户在对话中提供 **YApi 项目 id（`_yapi_uid`）** 与 **token（`_yapi_token`）**（及必要时 **YApi 服务根地址 base URL**），代理**仅将这些信息用于拉取 YApi 在线接口文档**；据此生成的是**业务后端接口**对应的 **`src/api/api.ts`**（业务请求层）与 **`src/models/model.ts`**，**不是**「调用 YApi 开放接口本身」的客户端代码。本能力为 **Skill**，不要实现为 Rule。

## 运行前提

- **在线拉取**：Agent 执行「根据 `_yapi_uid`、`_yapi_token`、（必要时）YApi `base URL` 拉取 YApi 文档」时，运行环境须具备**可用的网络访问**（能请求用户给出的 YApi 服务地址）。无网络时不要假设一定能在线拉取成功。
- **无法联网时**：向用户说明限制；尽量退化为由用户提供**其可访问的 YApi 导出内容**（如 JSON / Markdown 等粘贴或附件），再基于该内容生成代码。
- **首版定位不变**：Skill 的第一目标仍是「在线拉取 YApi 文档并生成业务 `api.ts` / `model.ts`」；导出内容仅为联网失败或不方便时的退化路径。

## YApi 鉴权优先级（拉取文档阶段）

- 当用户提供 `_yapi_uid` 与 `_yapi_token` 时，优先使用 **Cookie 鉴权** 拉取文档：`Cookie: _yapi_token=...; _yapi_uid=...`。
- 若用户明确指定其他鉴权方式（如 query token），按用户要求执行；否则保持 Cookie 优先。
- 上述鉴权信息仅用于「拉取 YApi 文档」，不得写入生成代码。

## 鉴权自动探测流程（拉取文档阶段）

1. 先用轻量接口做可用性探测（如 `project/get` 或 `interface/list`）。
2. 若返回登录态错误（如 `errcode=40011` / “请登录”），自动切换并重试 Cookie 鉴权。
3. 一旦某种方式成功，本次会话固定使用该方式继续分页拉取与详情拉取。
4. 若所有方式失败，向用户说明失败原因与已尝试方案，再退化为让用户提供可访问的导出文档内容。

## 第一版范围

**支持**

- 仅 **YApi** 开放平台/同源接口（鉴权后拉取项目与接口列表、单接口详情）。
- 用户 **仅在聊天中** 提供 `_yapi_uid`（项目 id）、`_yapi_token`（token）；可选提供 **YApi 根 URL**（如 `https://yapi.example.com`）。不要求编辑项目配置文件，不要求用户准备中间 JSON 文件。
- 输出路径固定：**`src/api/api.ts`**、**`src/models/model.ts`**（相对当前工作区/项目根）。

**不支持**

- 非 YApi 平台、Vue/页面代码、把本流程写成 Rule。

## 规范优先级

1. 若用户在本次对话中提供了 **Markdown 代码生成规范**，则 **优先** 按其执行（与下条冲突时以用户规范为准）。
2. 否则读取并遵循本 Skill 内置规范：**[references/default-codegen-spec.md](references/default-codegen-spec.md)**。

## 核心工作流

1. **收集上下文**（从当前聊天读取；缺失则简短追问一次）
   - 必填：`_yapi_uid`、`_yapi_token`
   - 强烈建议：`YApi base URL`（无协议前缀时补 `https://`）。若用户只说「内网地址」，仍须追问可请求的完整根 URL。
2. **拉取文档**（使用 `fetch` 或 axios；**依赖网络**；见上文「运行前提」）
   - **`_yapi_uid`、`_yapi_token`、YApi `base URL` 仅用于本步**，不得写入生成的业务 `api.ts`。
   - 鉴权执行遵循「Cookie 优先 + 自动探测」：默认优先 `_yapi_uid/_yapi_token` Cookie；失败时按探测流程回退。
   - 在 YApi `base URL` 上调用 YApi 开放接口（常见形态如下；若实例版本不同，以实际返回为准并容错）：
     - `GET /api/project/get?token={_yapi_token}` — 校验 token、拿项目元信息（请求参数名为 token；其值来自聊天字段 `_yapi_token`）
     - `GET /api/interface/list?project_id={_yapi_uid}&token={_yapi_token}&page={n}&limit={pageSize}` — 分页拉取接口列表（请求参数名为 project_id/token；其值来自聊天字段 `_yapi_uid`/`_yapi_token`），**循环直至无更多数据**
     - 对每条接口必要时 `GET /api/interface/get?id={interface_id}&token={_yapi_token}` — 补全 path、method、req_query、req_headers、req_body_type、req_body_form、req_body_other、res_body 等
   - 将原始 JSON **整理为结构化清单**：每个接口包含 method、path、标题、说明、path/query/body/form/header 参数、响应结构线索（JSON Schema / JSON 示例）。
3. **生成代码**（业务接口层，与 YApi 拉取凭证解耦）
   - 按规范文件生成 **`src/api/api.ts`**（业务后端请求层）与 **`src/models/model.ts`**。
   - 生成的 `api.ts` 面向 **YApi 文档中所描述的业务 API**（path/method/入参/响应）；**不要**把 YApi `token`、YApi 服务 `base URL`、YApi 开放接口的响应包裹形态默认写进该文件。
   - **默认 HTTP 客户端**：axios（用户明确要求其他封装时再换）。
4. **写入文件**
   - 覆盖或创建上述两个路径；若项目已有同名文件，合并策略：**以本次 YApi 文档为准完整重写该职责范围内的导出**（或用户另有说明时从其说明），避免半残混合 unless 用户要求增量。

## 生成物要点（摘要）

详细规则见 [references/default-codegen-spec.md](references/default-codegen-spec.md)。

- **api.ts**：TypeScript **业务请求层**；区分 path / query / body / form-data / headers；请求与响应类型；函数命名优先级：**可复用的 `operationId`** → **接口标题/摘要** → **`method + path` 语义**（均 camelCase，避免 `getData1` 等含糊名）；每接口简短注释（标题、Method、Path、说明）；尽量少 `any`。**不要**默认生成仅适用于 YApi 开放接口的 `YApiResponse<T>`；仅当业务文档明确统一响应包裹时才生成如 `ApiResponse<T>`。
- **model.ts**：`ModelBase`、`DataModel`、`Column` 均从 **`@model-base/core`** 导入；类名 **`XxxModel`**；属性与 TS 类型字段一律 **camelCase**。**仅风格差异**（camelCase vs snake_case）优先依赖 **ModelBase 官方命名策略与默认序列化/反序列化**，**不要**为每个字段机械加 `@Column({ name: '...' })`；**仅当后端字段名与前端属性是明确别名关系**（不仅是风格差异）时才使用 **`@Column({ name: 'raw_field_name' })`**。嵌套/数组/懒加载写法严格按官网（如 `@Column({ model: AddressModel })`、`@Column({ model: OrderItemModel, default: () => [] })`、`@Column({ model: () => ChildModel })`）。时间默认 **`string`**；**serialize / deserialize** 仅在用户明确要求且与官网一致时使用。**禁止**发明非官网的装饰器参数或映射约定。

## 降级与容错

文档缺失字段时：**不中断整次生成**；用 `optional`、`unknown`、或带 **`TODO:`** 的注释标明不确定处；仍输出尽可能可编译、可调用的代码。详见规范文件。

## 自动 / 手动触发

- **自动**：描述字段已包含英文与中文触发词（YApi、API 生成、api.ts、model.ts、请求层、鉴权文档等），相关意图下应优先匹配本 Skill。
- **手动**：用户可输入 **`yapi-doc-to-code`** 或本 Skill 名称以显式触发。

## 附加资源

- 默认代码生成细则：**[references/default-codegen-spec.md](references/default-codegen-spec.md)**
