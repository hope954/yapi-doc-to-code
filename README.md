# yapi-doc-to-code

基于聊天中提供的 YApi 鉴权信息，在线拉取接口文档并生成 TypeScript 请求层与模型定义。

## 触发词（示例）

- `YApi`
- `YApi 拉取`
- `YApi 鉴权文档`
- `生成 api`
- `自动生成 api`
- `生成 api.ts`
- `生成 model.ts`
- `请求层生成`
- `根据鉴权后的接口文档生成代码`
- `yapi-doc-to-code`（手动触发名）

## 输入参数

- `_yapi_uid`：YApi 项目 id
- `_yapi_token`：YApi 鉴权 token（可用于 Cookie 注入）
- `base URL`（可选但强烈建议）：YApi 服务根地址，例如 `http://127.0.0.1:3000`

说明：
- 拉取阶段默认优先使用 Cookie 鉴权：`_yapi_token` + `_yapi_uid`
- 若鉴权失败会按 Skill 规则做自动探测/回退

## 输出文件

- `src/api/api.ts`：业务接口请求层（默认 axios）
- `src/models/model.ts`：ModelBase 风格模型定义
