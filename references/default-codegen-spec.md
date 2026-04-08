# 默认代码生成规范（api.ts + model.ts）

本文件为 **yapi-doc-to-code** Skill 的默认规范。若用户在聊天中提供了另一份 Markdown 规范，则以用户规范为准（与本文件冲突时以用户为准）。

---

## 通用约定

- **语言**：TypeScript。
- **前端命名**：所有 **TypeScript 接口/类型字段**（含响应类型）、**请求参数对象字段**、**Model 类属性名** 一律 **camelCase**。**禁止**把前端属性写成 snake_case。
- **HTTP 客户端**：默认 **axios**。仅当用户明确要求（如 fetch、项目封装 `request`）时才替换；替换时仍须满足本文件对类型与参数分类的要求。

### 拉取 YApi 文档 vs 生成业务 `api.ts`（必须区分）

| 阶段 | 用途 | 是否进入生成代码 |
|------|------|------------------|
| Agent 拉取文档 | 使用聊天中的 YApi `_yapi_uid`（项目 id）、`_yapi_token`（token）、YApi `base URL` 调用 YApi **开放接口**（如 `project/get`、`interface/list`、`interface/get`） | **否**：这些是元数据拉取，**不得**写入生成的 `src/api/api.ts` |
| 生成 `api.ts` | 面向 YApi 文档里描述的 **业务 HTTP API**（真实后端的 path、method、鉴权与响应） | **是**：`base URL` 指向 **业务后端**（可用占位常量 + 环境变量说明），**不要**默认使用 YApi 服务地址；**不要**把「拉取文档用的 token」当作业务接口的默认鉴权写死 |

- **禁止**在生成的业务 `api.ts` 中默认引入 **`YApiResponse<T>`** 或等价、仅适用于 **YApi 开放接口返回** 的包裹类型。
- **仅当业务接口文档**（YApi 中该接口或项目级说明）**明确描述**统一响应包裹（如 `code`/`msg`/`data`）时，才可生成 **`ApiResponse<T>`**（或文档命名的等价泛型），字段名与文档一致。

### YApi 文档拉取鉴权（仅拉取阶段）

- 若聊天中提供 `_yapi_uid` 与 `_yapi_token`，默认优先按 **Cookie** 注入：`Cookie: _yapi_token=...; _yapi_uid=...`。
- 可先用轻量接口探测鉴权有效性；若出现登录态错误（如 `errcode=40011` / “请登录”），应自动回退到 Cookie 重试并在成功后固定本次会话的拉取方式。
- 该鉴权策略**仅用于拉取 YApi 文档元数据**，不得外溢到生成的业务请求层代码。

---

## 第一部分：`src/api/api.ts`（业务后端请求层）

### 1.1 模块结构

- 使用 **`import axios` 与 `AxiosInstance`、`AxiosRequestConfig`**（或等价类型）。
- 导出指向 **业务后端** 的 **可配置 base URL**（如 `API_BASE_URL` / `BUSINESS_API_BASE_URL`）及 **`axios.create` 实例**（如 `apiClient`），便于用户接环境变量；**不要**把 YApi 文档拉取用的 **`base URL` 当作业务 API 的默认 base**。
- **不要**在生成的业务 `api.ts` 中写死 **YApi `token`**、**YApi 项目鉴权** 或任何「仅用于拉取 YApi 文档」的常量；业务鉴权按 **业务文档**（如 Bearer、Cookie、业务 `token` 参数）生成，由调用方传入或拦截器扩展。
- **业务接口的响应类型**：直接按文档中的 **业务 JSON** 建模；无统一包裹时不要臆造全局 `data` 包裹。**不要**默认套用 YApi 开放接口的 `{ errcode, errmsg, data }` 形态。

### 1.2 每个接口必须包含

1. **请求参数类型**（若无可省略或使用空对象类型）
   - **Path params**：合并进参数类型，在 URL 模板中替换（如 `` `/api/user/${id}` ``）。
   - **Query**：`params` 对象；字段 camelCase；若文档要求固定 snake 的 query 名，在发送层使用文档名（与 axios `params` 键名一致）。
   - **JSON Body**：单独类型；`post(..., body)`。
   - **form-urlencoded / form-data**：类型仍 camelCase；序列化时在 **请求构造** 中转为文档要求的字段名（可配合 Model 的 `getSerializableObject()` 等官方 API，见下文 Model 节）。
   - **Headers**：在 `AxiosRequestConfig['headers']` 中合并；敏感头不写死默认值。

2. **响应类型**：`Promise<...>`，尽量贴合文档中的 JSON 结构；能对应到 Model 时可用 Model 类型或等价的 interface。

3. **请求函数**：
   - **命名优先级**（均转为合法 **camelCase**，可读、稳定、语义清晰；**禁止** `getData1` 类含糊命名）：
     1. 文档中有明确且可复用的 **`operationId`** 时 **优先**使用；
     2. 无 `operationId` 时，**优先**根据接口 **标题 / 摘要** 生成语义化名称；
     3. 标题缺失或不稳定时，再用 **`method` + `path` 的语义组合** 生成名称（可结合路径资源名、动作动词）。
   - **签名**：`(params: XxxParams, config?: AxiosRequestConfig) => Promise<...>`；无参接口用 `void` 或可选空对象。

4. **注释**（每个函数上方块注释或 JSDoc）：
   - 接口标题或摘要、**HTTP Method**、**Path**、简要说明（来自 YApi 描述）。

### 1.3 尽量少用 `any`

- 结构未知时用 **`unknown`**、**索引签名**、或 **带 `TODO:` 的 interface**。
- 列表元素未知时用 **`unknown[]`** 或 `Array<Record<string, unknown>>`。

### 1.4 与 Model 的协作

- 当 body 为 form 且项目采用 ModelBase 序列化时，可 **`XxxFormModel.create({ ... })`** 再 **`getSerializableObject()`** 写入 `URLSearchParams` 或 `FormData`，与现有 @model-base/core 用法一致。

---

## 第二部分：`src/models/model.ts`

### 2.1 导入与基类

```ts
import { Column, DataModel, ModelBase } from '@model-base/core';
```

- 每个领域模型类：**`@DataModel()`** 装饰器 + **`extends ModelBase`**。
- 类名统一 **`XxxModel`**（如 `UserDetailModel`、`OrderItemModel`）。

### 2.2 camelCase 属性与后端字段名（严格对齐官网语义）

- **统一规则**：前端 **TypeScript 类型字段**、**请求参数对象字段**、**响应类型字段**、**Model 属性名** 一律 **camelCase**；**禁止**用 snake_case 作为属性名。
- **仅有命名风格差异**（如后端 `user_name` ↔ 前端 `userName`）：**优先依赖 @model-base/core 官方命名策略** 与 **默认序列化 / 反序列化行为**，**不要**仅为风格转换而给每个字段加 **`@Column({ name: 'snake_case' })`**。
- **显式 `name` 的使用条件**：**仅当**后端字段名与前端属性名是 **明确别名关系**（**不仅是** camelCase/snake_case 风格差异），例如 **`_id` → `id`**、`userID` ↔ `user_id` 等文档或示例可证实的非对称命名时，才使用 **`@Column({ name: 'raw_field_name' })`**（或官网文档中与 `name` 等价的官方选项）。不得臆造 `name`。
- **禁止**发明与官网不一致的「全局映射约定」或自定义装饰器参数形式。

### 2.3 字段装饰器（必须与 @model-base/core 官网写法一致）

- **基础 / 标量 / 普通结构字段**：使用 **`@Column()`**（无额外参数，除非官网文档要求或符合 2.2 的显式 `name` 等官方选项）。
- **嵌套模型（单个对象）**：使用官网形式，例如  
  **`@Column({ model: AddressModel })`**。
- **嵌套模型数组**：使用官网形式，例如  
  **`@Column({ model: OrderItemModel, default: () => [] })`**（数组须配合官网推荐的 **`default`**，避免 `undefined` 与序列化歧义；以当前版本文档为准）。
- **循环引用或延迟求值**：使用与官网一致的懒加载形式，例如  
  **`@Column({ model: () => ChildModel })`**；数组场景下懒加载子模型时同样采用 **`model: () => Model`** 的官网写法。
- **时间 / 日期**：默认类型为 **`string`**（如 ISO 8601 字符串），除非用户明确要求其他表示方式。

### 2.4 禁止事项

- **不要**发明官网未记载的自定义装饰器参数、非官方命名映射规则或非官方转换模式。
- **不要**把 **属性名** 写成 snake_case；合法映射仅通过 **官方支持的 `Column` 配置**（如经 2.2 判断需要的 `name`）完成。

### 2.5 序列化 / 反序列化

- 生成代码须 **兼容 ModelBase 默认序列化、反序列化与命名策略**。
- **`serialize` / `deserialize` 等扩展**：**仅当用户在对话中明确要求**某字段需要自定义转换时，才采用与 **@model-base/core 官方文档一致** 的写法；不得默认加钩子。其他覆盖项也仅限 **官方文档明确记载** 的 API（以实际安装的 `@model-base/core` 版本为准）。

### 2.6 请求体 / 响应体与类的对应关系

- 为 **可识别的 JSON 对象**、**明确嵌套结构** 生成独立 **Model 类**。
- 同一结构多处复用时 **复用同一 Model**，避免重复类。
- 若文档仅有示例 JSON 无 Schema，**推断合理字段**；不确定处加 **`TODO:`** 注释或 **`unknown`**。

---

## 第三部分：文档不完整时的降级策略

0. **无法在线拉取时**（环境无网络等）：不假设拉取一定成功；由用户粘贴或提供 **YApi 导出内容** 后再生成（首版仍以在线拉取为主，见 SKILL.md「运行前提」）。
1. **不要中断**：缺字段、缺 Schema、部分接口无 response 示例时，仍生成其余接口的完整代码。
2. **参数/响应**：缺失时用 **可选字段 `?`**；结构不明用 **`unknown`** 或 **`Record<string, unknown>`**。
3. **TODO**：对无法确认的语义、枚举值、联合类型，用 **`// TODO: ...`** 标明假设或待核对点。
4. **命名冲突**：YApi 标题重复时，函数名加路径或分类前缀区分。
5. **分页列表**：若只拉到部分接口，在文件头注释说明「基于当前拉取结果生成」，并建议用户检查分页是否完整。

---

## 第四部分：不确定字段的 TODO 策略

- **单字段不确定**：属性旁或列上注释 `TODO: YApi 未描述类型，暂为 string`。
- **整块结构不确定**：使用 `unknown` 或空接口扩展，并 `TODO` 说明待补全。
- **枚举**：文档未列枚举时，用 **`string`** 或 **`string` + TODO**，不臆造字面量联合。

---

## 第五部分：与 YApi 字段的对应关系（参考）

代理从 YApi 接口详情中读取的常见字段（名称以实例为准）：

| 概念 | 生成时的去向 |
|------|----------------|
| `method` + `path` | axios method + URL |
| `req_params` / path | 路径替换与参数类型 |
| `req_query` | `params` |
| `req_headers` | `headers` 类型与传入 |
| `req_body_type` json | JSON body + 类型 |
| `req_body_type` form / raw | form / 原始 body 处理 |
| `req_body_form` | 表单字段模型 |
| `res_body` / mock | 响应类型与 Model |

以上为生成时的**逻辑映射**，实际字段名以拉取到的 JSON 为准。
