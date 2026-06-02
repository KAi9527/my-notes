该版本修改deploy start行为，触发部署后等待终态返回；log search添加--live选项查询实时日志接口

# JoyGen 沙盒部署验证规约  
  
本规约用于编码助手在代码修改完成后，通过 JoyGen DevTool CLI 完成沙盒部署、构建日志分析与运行日志观察，建立“编码 -> 部署 -> 日志诊断 -> 修改”的反馈循环。  
  
## 触发时机  
  
当编码助手对可运行应用代码、构建配置、依赖配置、运行时配置、数据库/中间件访问逻辑、接口逻辑或前端页面行为做出修改后，应进入沙盒验证流程。  
  
以下情况可以跳过沙盒部署，但需要在最终回复中说明原因：  
  
- 仅修改文档、注释、README、示例文本。  
- 仅做静态重命名或格式化，且不影响构建产物和运行时行为。  
- 用户明确要求不要部署或只做本地修改。  
- 当前环境缺少沙盒部署触发能力、鉴权信息或项目标识，且编码助手无法自行补齐。  
  
## CLI 基础能力  
  
JoyGen DevTool CLI 应在 Node 服务端环境中预先全局安装：  
  
```bash  
npm install -g @jdei/devtool-cli@latest```  
  
沙盒链路提供以下可以独立调用的基础能力：  
  
| 能力 | 命令 | 作用 |  
|---|---|---|  
| 触发部署 | `joygen-devtool deploy start --env dev --output json` | 触发当前项目沙盒部署，sandbox 路由下等待 ready/failed 后返回，并启动 deploy 守护线程 |  
| 构建日志 | `joygen-devtool logs search --env dev --live build --limit 200 --output json` | 部署失败后绕过本地 daemon 获取构建日志，读取 `data.build.logs` |  
| 运行日志 | `joygen-devtool logs search --env dev --live running --limit 200 --output json` | 部署成功后绕过本地 daemon 获取运行日志，读取 `data.running.logs` |  
  
沙盒资源、鉴权和 RUNCFG 参数由 Node 服务端运行环境通过 `JOYGEN_DEVTOOL_*` 环境变量注入。  
  
## 必需配置  
  
沙盒 CLI 调用需要以下环境变量：  
  
```bash  
export JOYGEN_DEVTOOL_PROJECT=<project-name>export JOYGEN_DEVTOOL_PROJECT_PROJECTID=<project-id>export JOYGEN_DEVTOOL_PROJECT_TASKID=<task-id>export JOYGEN_DEVTOOL_USER_COOKIE=<cookie>export JOYGEN_DEVTOOL_USER_USERNAME=<erp>export JOYGEN_DEVTOOL_DEPLOY_PROVIDER=sandboxexport JOYGEN_DEVTOOL_DEPLOYSTAT_PROVIDER=sandboxexport JOYGEN_DEVTOOL_LOGQUERY_PROVIDER=sandboxexport JOYGEN_DEVTOOL_OUTPUT_FORMAT=jsonexport JOYGEN_DEVTOOL_DEPLOY_WAIT_TIMEOUT_MS=300000```  
  
其中 `JOYGEN_DEVTOOL_USER_COOKIE` 等敏感信息不得写入最终回复或报告摘要。  
  
## 标准流程  
  
1. 完成代码修改。  
2. 确认服务端执行环境已安装 `joygen-devtool`，并已注入项目、鉴权、provider 路由和必要 RUNCFG 变量。  
3. 使用 `joygen-devtool deploy start --env dev --output json` 触发沙盒部署，并等待命令返回。  
4. 若部署失败，调用 `joygen-devtool logs search --env dev --live build --limit 200 --output json` 获取构建日志，读取 `data.build.logs` 的 200 行用于分析原因。  
5. 若部署成功，每隔 8 秒调用 `joygen-devtool logs search --env dev --live running --limit 200 --output json` 获取运行日志，读取 `data.running.logs` 的 200 行观察运行情况。  
6. 若运行日志显示应用启动失败，基于日志分析原因并修改代码，然后重新从 `deploy start` 开始。  
7. 若运行日志显示应用启动成功，本次沙盒反馈操作完成，记录 `sandboxId`、`runId`、`serviceUrl`、`traceId`、`bCode` 等可用排障字段。  
  
`deploy start` 默认等待超时时间为 5 分钟；若存在 `JOYGEN_DEVTOOL_DEPLOY_WAIT_TIMEOUT_MS`，则以该环境变量的值为准，单位为毫秒。  
  
日志查询参数仅调整 `--live build|running` 与 `--limit 200`；除这两个选项外，其余参数和选项保持既有调用上下文，不新增关键词、级别、时间窗口或 offset 等筛选条件。  
  
## 判定标准  
  
### 部署结果判定  
  
部署成功需要同时满足：  
  
- `deploy start` 命令退出码为 0。  
- stdout 可解析为结构化 JSON。  
- 结构化结果 `status=SUCCESS`。  
- sandbox 路由已等待到 ready 结果。  
  
以下情况均视为部署失败：  
  
- sandbox 返回 `failed=true`。  
- 结构化结果 `status=FAILED` 或 `status=PARTIAL`。  
- 命令退出码非 0。  
- 等待 ready/failed 超时。  
- stdout 不是可解析 JSON。  
- 结果缺少足以判定 ready 或 failed 的字段。  
  
### 运行结果判定  
  
运行成功需要同时满足：  
  
- `logs search --live running --limit 200` 命令退出码为 0，且 stdout 可解析为结构化 JSON。  
- 运行日志读取 `data.running.logs`，不依赖顶层 `data.logs`。  
- 运行日志中出现应用启动完成、服务监听、ready、healthy 等启动成功信号。  
- 运行日志中没有启动失败、未捕获异常、端口绑定失败、依赖连接失败、配置缺失、进程退出等失败信号。  
  
运行失败包括但不限于：  
  
- 运行日志明确出现启动失败、进程退出、未捕获异常或依赖初始化失败。  
- 多次每隔 8 秒查询运行日志后仍没有启动成功信号，且日志持续显示阻塞、重试或异常。  
- `logs search --live running --limit 200` 返回失败、超时或不可解析 JSON。  
  
若只能完成状态或日志查询但没有部署触发动作，不能判定为“部署验证通过”，只能判定为“观测当前沙盒状态”。  
  
## 输出要求  
  
最终回复必须说明：  
  
- 沙盒部署是否实际触发。  
- 部署结果摘要。  
- 构建日志或运行日志检查摘要。  
- 关键排障字段：`sandboxId`、`runId`、`serviceUrl`、`traceId`、`bCode`，若字段不存在则说明未返回。