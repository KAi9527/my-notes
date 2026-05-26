该版本为多步骤连续执行版本

---  
name: "joygen-sandbox-feedback-loop"  
description: "当编码 Agent 完成应用代码修改后，需要触发 JoyGen 沙盒部署、查询沙盒状态和日志，并基于结果继续调试时使用。"  
---  
  
# JoyGen Sandbox Feedback Loop  
  
用于企业编码 Agent 在改完代码后执行沙盒部署验证闭环。  
  
## 关键事实  
  
- 企业编码 Agent 在 Node 服务端直接执行全局安装的 `joygen-devtool`。  
- 服务端应预先执行 `npm install -g @jdei/devtool-cli@latest`，让 Agent 能直接使用命令行工具。  
- Agent 不通过 MCP 调用 CLI；沙盒部署、状态和日志验证都通过 CLI 完成。  
- `joygen-devtool deploy start` 已开放，优先用它触发 sandbox 部署。  
- sandbox 路由下 `deploy start` 成功后会启动 deploy 守护线程，后续用 `status app` 和 `logs search` 验证。  
- Claude Code 非交互模式下通过项目级 `SessionEnd` hook 自动执行部署和日志采集。  
- `SessionEnd` hook 发现错误时会输出报告摘要，并默认用阻断退出码要求支持该机制的 Agent/Claude Code hook runner 继续排错。  
- 如运行环境不支持阻断续跑，错误会写入报告，下一轮应读取报告继续修复。  
  
## 何时使用  
  
在完成影响应用运行行为的代码修改后使用，包括：  
  
- 后端接口、服务逻辑、数据库访问、中间件访问。  
- 前端页面、交互、构建产物。  
- 依赖、构建脚本、运行时配置。  
- 修复线上或沙盒日志中暴露的问题。  
  
文档、注释、纯格式化修改可跳过，但最终回复要说明跳过原因。  
  
## 环境变量  
  
CLI 观测需要：  
  
```bash  
export JOYGEN_DEVTOOL_PROJECT=<project-name>export JOYGEN_DEVTOOL_PROJECT_PROJECTID=<project-id>export JOYGEN_DEVTOOL_USER_COOKIE=<cookie>export JOYGEN_DEVTOOL_USER_USERNAME=<erp>export JOYGEN_DEVTOOL_DEPLOY_PROVIDER=sandboxexport JOYGEN_DEVTOOL_DEPLOYSTAT_PROVIDER=sandboxexport JOYGEN_DEVTOOL_LOGQUERY_PROVIDER=sandboxexport JOYGEN_DEVTOOL_OUTPUT_FORMAT=json```  
  
## 标准执行流程  
  
1. 先完成代码修改。  
2. 跑项目已有的最小本地验证。优先顺序通常是类型检查、构建、单测、lint；按项目现有脚本选择。  
3. 确认当前 Node 服务端环境可以执行 `joygen-devtool --version`，并已注入必要 `JOYGEN_DEVTOOL_*` 环境变量。  
4. 调用 CLI 触发部署：  
  
```bash  
joygen-devtool deploy start --env dev --output json```  
  
5. 部署触发成功后执行：  
  
```bash  
joygen-devtool status app --env dev --output json```  
  
6. 等待 5 到 8 秒，再执行：  
  
```bash  
joygen-devtool logs search --env dev --limit 80 --output json```  
  
7. 如需定位错误，再执行：  
  
```bash  
joygen-devtool logs search --env dev --keyword ERROR --limit 80 --output json```  
  
8. 如果状态或日志失败，基于日志修复代码，然后重新从本地验证开始。  
  
## SessionEnd Hook  
  
项目级 hooks 文件：  
  
```text  
.claude/settings.json  
.claude/hooks/joygen-sandbox-session-end.sh  
```  
  
hook 会在会话结束时执行：  
  
```text  
deploy start -> status app -> logs search -> logs search --keyword ERROR  
```  
  
报告目录：  
  
```text  
.claude/joygen-sandbox-reports/latest/report.md  
```  
  
可选控制变量：  
  
```bash  
export JOYGEN_SANDBOX_HOOK_ENABLED=0          # 禁用 hookexport JOYGEN_SANDBOX_HOOK_ALWAYS_RUN=1       # 即使未检测到代码变更也运行export JOYGEN_SANDBOX_HOOK_ENV=dev            # 覆盖部署环境export JOYGEN_SANDBOX_HOOK_WAIT_SECONDS=8     # deploy 后等待秒数export JOYGEN_SANDBOX_HOOK_LOG_LIMIT=120      # 日志条数export JOYGEN_SANDBOX_HOOK_DEPLOY_VERSION=... # 可选版本参数export JOYGEN_SANDBOX_HOOK_BLOCK_ON_ERROR=0   # 只记录报告，不用阻断退出码要求续跑```  
  
## 结果判定  
  
可以判定沙盒验证通过的条件：  
  
- 部署触发动作成功。  
- `status app` 返回结构化成功结果。  
- 应用健康状态为 healthy，或返回的 serviceAvailable 为 true。  
- 聚合日志中没有与本次修改相关的构建失败、启动失败、运行时 ERROR。  
  
不能判定通过的情况：  
  
- 只查询了旧沙盒状态，没有触发新部署。  
- `deploy start` 返回失败。  
- 日志只返回空结果且状态未 ready。  
  
## 输出给用户  
  
最终回复要包含：  
  
- 本地验证结果。  
- 是否实际触发沙盒部署。  
- `status app` 摘要。  
- `logs search` 摘要。  
- 关键字段：`sandboxId`、`runId`、`serviceUrl`、`traceId`、`bCode`。  
  
## Guardrails  
  
- 优先用 `joygen-devtool deploy start` 触发部署，再查询状态和日志。  
- 不要输出 cookie、token、资源密码等敏感信息。  
- 不要假设 MCP 参与部署链路；本技能的部署、状态、日志操作均通过 CLI 完成。  
- `daemon run` 是内部命令，除排障外不要人工调用。