该版本为多步骤连续执行版本

# JoyGen 沙盒部署验证规约  
  
本规约用于企业编码 Agent。目标是在代码修改完成后，自动触发 JoyGen 沙盒部署，并通过状态与日志查询形成“编码 -> 部署 -> 验证 -> 调试 -> 修改”的反馈循环。  
  
## 触发时机  
  
当 Agent 对可运行应用代码、构建配置、依赖配置、运行时配置、数据库/中间件访问逻辑、接口逻辑或前端页面行为做出修改后，必须进入沙盒验证流程。  
  
以下情况可以跳过沙盒部署，但需要在最终回复中说明原因：  
  
- 仅修改文档、注释、README、示例文本。  
- 仅做静态重命名或格式化，且不影响构建产物和运行时行为。  
- 用户明确要求不要部署或只做本地修改。  
- 当前环境缺少沙盒部署触发能力、鉴权信息或项目标识，且 Agent 无法自行补齐。  
  
## 能力边界  
  
企业编码 Agent 通过 Node 服务端环境中全局安装的 CLI 直接执行沙盒操作。服务端应预先安装：  
  
```bash  
npm install -g @jdei/devtool-cli@latest```  
  
Agent 不通过 MCP 调用 CLI，也不依赖 MCP 触发部署。JoyGen DevTool CLI 当前提供沙盒部署、状态与日志观测能力：  
  
- `joygen-devtool deploy start --env dev --output json`  
- `joygen-devtool status app --env dev --output json`  
- `joygen-devtool logs search --env dev --limit 50 --output json`  
- `joygen-devtool logs search --env dev --keyword ERROR --limit 50 --output json`  
  
沙盒部署触发应使用 `joygen-devtool deploy start`。沙盒资源、鉴权和 RUNCFG 参数应由 Node 服务端运行环境通过 `JOYGEN_DEVTOOL_*` 环境变量注入。  
  
## 必需配置  
  
沙盒观测 CLI 需要以下环境变量：  
  
```bash  
export JOYGEN_DEVTOOL_PROJECT=<project-name>export JOYGEN_DEVTOOL_PROJECT_PROJECTID=<project-id>export JOYGEN_DEVTOOL_USER_COOKIE=<cookie>export JOYGEN_DEVTOOL_USER_USERNAME=<erp>export JOYGEN_DEVTOOL_DEPLOY_PROVIDER=sandboxexport JOYGEN_DEVTOOL_DEPLOYSTAT_PROVIDER=sandboxexport JOYGEN_DEVTOOL_LOGQUERY_PROVIDER=sandboxexport JOYGEN_DEVTOOL_OUTPUT_FORMAT=json```  
  
## 标准流程  
  
1. 完成本地代码修改。  
2. 运行项目已有的最小本地验证，例如类型检查、构建、单测或 lint。若本地验证失败，先修复本地问题，不进入沙盒部署。  
3. 确认 Node 服务端执行环境已安装 `joygen-devtool`，并已注入项目、鉴权、provider 路由和必要 RUNCFG 变量。  
4. 使用 `joygen-devtool deploy start --env dev --output json` 触发沙盒部署。  
5. 部署触发成功后，调用 `joygen-devtool status app --env dev --output json` 获取首个 sandbox 快照。  
6. 等待 5 到 8 秒后，调用 `joygen-devtool logs search --env dev --limit 80 --output json` 获取构建日志与运行时日志。  
7. 若状态为 ready/healthy 且日志无明显错误，记录 `sandboxId`、`runId`、`serviceUrl`、`traceId`、`bCode`。  
8. 若状态失败、超时或日志包含错误，优先根据 `source=sandbox` 的构建日志和 `source=runtime` 的运行时日志定位问题，修改代码后重新进入本流程。  
  
## 判定标准  
  
沙盒验证通过需要同时满足：  
  
- 沙盒部署触发动作成功返回。  
- `status app` 返回 `status=SUCCESS`。  
- `data.health` 为 `healthy`，或 `meta/serviceAvailable` 表示服务可用。  
- `logs search` 中不存在与本次修改相关的 `ERROR`、启动失败、依赖连接失败、构建失败、接口异常等日志。  
  
若只能完成状态/日志查询但没有部署触发动作，不能判定为“部署验证通过”，只能判定为“观测当前沙盒状态”。  
  
## 输出要求  
  
最终回复必须说明：  
  
- 本地验证命令及结果。  
- 沙盒部署是否实际触发。  
- 沙盒状态摘要。  
- 日志检查摘要。  
- 关键排障字段：`sandboxId`、`runId`、`serviceUrl`、`traceId`、`bCode`，若字段不存在则说明未返回。  
  
## SessionEnd Hook  
  
Claude Code 非交互模式下应使用项目级 `SessionEnd` hook 自动执行上述 CLI。hook 配置位于：  
  
```text  
.claude/settings.json  
.claude/hooks/joygen-sandbox-session-end.sh  
```  
  
hook 会在会话结束时自动执行：  
  
```text  
deploy start -> status app -> logs search -> logs search --keyword ERROR  
```  
  
报告输出到：  
  
```text  
.claude/joygen-sandbox-reports/latest/report.md  
```  
  
当部署、状态或错误日志检查失败时，hook 会把报告摘要写入 stderr，并默认以阻断退出码反馈给支持该机制的 Agent/Claude Code hook runner，要求继续读取报告并排错。若运行环境只希望记录报告、不阻断会话结束，可设置：  
  
```bash  
export JOYGEN_SANDBOX_HOOK_BLOCK_ON_ERROR=0```  
  
若 hook runner 不支持阻断续跑，下一轮 Agent 应优先读取上述报告继续修复。