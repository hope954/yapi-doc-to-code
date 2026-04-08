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
- `url`（可选但强烈建议）：YApi 具体页面路径，例如 `http://127.0.0.1:3000/project/11/interface/api/11`

说明：
- 拉取阶段默认优先使用 Cookie 鉴权：`_yapi_token` + `_yapi_uid`
- 若鉴权失败会按 Skill 规则做自动探测/回退

## 输出文件

- `src/api/api.ts`：业务接口请求层（默认 axios）
- `src/models/model.ts`：ModelBase 风格模型定义

## 最小对话示例

```text
用 yapi-doc-to-code 生成请求层和类型定义。
url: http://127.0.0.1:3000/project/11/interface/api/11
_yapi_uid: 11
_yapi_token: <your-token>
请生成 src/api/api.ts 和 src/models/model.ts
```

## 常见失败与排查

- **返回“请登录”或 `errcode=40011`**：优先检查 `_yapi_uid` / `_yapi_token` 是否正确、是否过期。
- **页面 URL 无法访问**：请确认你提供的是可直接访问的完整页面路径（如 `/project/.../interface/api/...`），并且当前环境可访问该地址。
- **内网地址不可达**：当前环境无法访问你的 YApi 服务时，先确认网络连通；必要时改为提供可访问的导出文档内容。
