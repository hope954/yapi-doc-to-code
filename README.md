# yapi-doc-to-code

基于聊天中提供的 YApi 鉴权信息，在线拉取接口文档并生成 TypeScript 请求层与模型定义。

## 能力分层

- 当前项目仅保留 `v1` 与 `v3` 两个能力层级。
- `v1`：根据受权限保护的 API 文档，生成 `src/api/api.ts` 与 `src/models/model.ts`。
- `v3`：结合 PRD（PDF + 飞书在线文档）+ Figma + `v1` 结果，先生成 `page.manifest.json` 与 `cursor-page-prompt.md`，再由 Cursor 生成页面代码。
- `v2`：不再保留。

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

## 安装与使用（Cursor）

### 方式一：项目级使用（推荐随仓库分发）

适合希望“克隆项目后即可在该项目中使用”。

1. 克隆仓库后，用 Cursor 打开项目根目录。
2. 确认目录结构为：`.cursor/skills/yapi-doc-to-code/SKILL.md`。
3. 在 Cursor 中执行一次 `Developer: Reload Window`（或重启 Cursor）。
4. 新建一个 Agent 会话，即可通过触发词调用本 Skill。

说明：
- 项目级 Skill 仅在“当前打开的这个项目”中可见和可用。
- 如果打开的是其他目录（不是该仓库根目录），Cursor 可能无法扫描到本 Skill。

### 方式二：全局级使用（所有项目可用）

适合希望“任意项目都能使用这个 Skill”。

1. 将 `yapi-doc-to-code` 整个目录复制到本机 Cursor 全局技能目录：
   - Windows：`C:\Users\<你的用户名>\.cursor\skills\yapi-doc-to-code`
   - macOS/Linux：`~/.cursor/skills/yapi-doc-to-code`
2. 确认目录内存在 `SKILL.md`。
3. 重载 Cursor 窗口并新建 Agent 会话。

说明：
- 全局级安装后，无需依赖某个特定仓库目录即可使用。

#### 一键全局安装命令

下面命令会直接安装到本机 Cursor 用户目录（不是某个项目目录）。

**Windows（PowerShell）**

```powershell
$target="$env:USERPROFILE\.cursor\skills\yapi-doc-to-code"
if (Test-Path $target) { Remove-Item $target -Recurse -Force }
git clone https://github.com/hope954/yapi-doc-to-code.git $target
```

**macOS / Linux（bash）**

```bash
target="$HOME/.cursor/skills/yapi-doc-to-code"
rm -rf "$target"
git clone https://github.com/hope954/yapi-doc-to-code.git "$target"
```

## 安装 Figma MCP

1. 在 Cursor 聊天中执行命令：`/add-plugin figma`。
2. 打开 Cursor `Settings -> Tools & MCP`，找到 Figma 插件并完成连接与授权。
3. 授权完成后，建议先做一次连通性测试。

中文测试提示词示例：

```text
请读取我已连接的 Figma 文件，列出当前页面的主要组件结构，并给出颜色、字号、间距的设计要点摘要。
```

## 安装飞书/Lark MCP 与飞书在线文档权限配置

1. 先安装 Node.js（建议使用 LTS 版本），并确认本机命令可用：
   - `node -v`
   - `npm -v`
2. 前往飞书/Lark 开放平台创建应用，获取 `App ID` 与 `App Secret`。
3. 若需要读取私有飞书在线文档，请在应用配置中设置 OAuth 回调地址为：`http://localhost:3000/callback`。
4. 使用登录命令完成本地 OAuth 登录（将占位符替换为你的应用信息）：
   - `npx @larksuiteoapi/lark-mcp login --app-id <APP_ID> --app-secret <APP_SECRET> --redirect-uri http://localhost:3000/callback`
5. 在 Cursor 的 MCP 配置中添加飞书/Lark 服务，示例如下：

```json
{
  "mcpServers": {
    "lark": {
      "command": "npx",
      "args": [
        "@larksuiteoapi/lark-mcp",
        "--oauth",
        "--token-mode",
        "user_access_token"
      ]
    }
  }
}
```

说明：
- 推荐在应用权限（scope）中启用：`offline_access`、`docx:document`。
- 飞书在线文档在本项目中主要作为只读 PRD 输入源，用于 `v3` 阶段的信息整合与页面生成准备。

## page.manifest.json 是什么

- `page.manifest.json` 是页面生成流程中的结构化中间结果。
- 该文件用于融合 API / PRD / Figma 三类输入信息，形成统一的页面实现描述。
- 第一版流程中至少保留该文件，便于复核、迭代与二次生成。
- 文件内容通常可能包含：
  - 页面类型
  - 模块名
  - API 绑定
  - 字段分组
  - 组件拆分建议
  - TODO 列表

## cursor-page-prompt.md 是什么

- `cursor-page-prompt.md` 是提供给 Cursor 的页面生成提示词文件。
- 用户可以在生成前或生成后继续手动修改该文件内容。
- 最终由 Cursor 根据该文件生成或修改页面代码。

## 页面代码组织方式

- 默认实现策略偏向“按功能拆组件”，而不是把全部逻辑集中在单个 `index.vue`。
- 参考目录示例：
  - `src/views/<module>/index.vue`
  - `src/views/<module>/components/SearchForm.vue`
  - `src/views/<module>/components/DataTable.vue`
  - `src/views/<module>/components/EditDialog.vue`
  - `src/views/<module>/components/DetailDrawer.vue`

## UI 库选择策略

- 用户已明确指定 UI 库时，优先使用用户指定方案。
- 用户未指定时，先检查项目依赖中已存在的 UI 库并优先复用。
- 若项目依赖中均不存在，再按 `Arco > Element > Ant` 的顺序选择。
- 若最终选中的 UI 库尚未安装，需要先安装对应依赖后再生成页面代码。
