
该版本修改deploy start行为，触发部署后等待终态返回；log search添加--live选项查询实时日志接口

---  
name: "joygen-sandbox-feedback-loop"  
description: "当编码助手需要使用 JoyGen 沙盒进行部署、构建日志分析或运行日志观察时使用。"  
---  
  
# JoyGen Sandbox Feedback Loop  
  
用于编码助手在代码修改后通过 `joygen-devtool` 触发 JoyGen 沙盒部署，并根据构建日志或运行日志完成反馈闭环。  
  
## CLI 基础能力  
  
`joygen-devtool` 提供三类可以单独调用的沙盒能力。  
  
### 触发部署  
  
```bash  
joygen-devtool deploy start --env dev --output json```  
  
作用：  
  
- 触发当前项目的沙盒部署。  
- sandbox 路由下会等待 ready/failed 后返回，并启动 deploy 守护线程。  
- 默认等待超时时间为 5 分钟；若存在 `JOYGEN_DEVTOOL_DEPLOY_WAIT_TIMEOUT_MS`，则以该环境变量的值为准，单位为毫秒。  
- 可选传入版本：  
  
```bash  
joygen-devtool deploy start --env dev --ver <version> --output json```  
  
### 构建日志  
  
```bash  
joygen-devtool logs search --env dev --live build --limit 200 --output json```  
  
作用：  
  
- 部署失败后拉取构建日志。  
- 读取 `data.build.logs`，每次最多获取 200 行用于分析原因。  
  
### 运行日志  
  
```bash  
joygen-devtool logs search --env dev --live running --limit 200 --output json```  
  
作用：  
  
- 部署成功后拉取运行日志。  
- 读取 `data.running.logs`，每次最多获取 200 行用于观察运行情况。  
  
live 模式日志不要读取顶层 `data.logs`。  
  
## 何时使用  
  
在完成影响应用运行行为的代码修改后使用，包括：  
  
- 后端接口、服务逻辑、数据库访问、中间件访问。  
- 前端页面、交互、构建产物。  
- 依赖、构建脚本、运行时配置。  
- 修复沙盒日志中暴露的问题。  
  
文档、注释、纯格式化修改可跳过，但最终回复要说明跳过原因。  
  
## 环境变量  
  
CLI 调用需要：  
  
```bash  
export JOYGEN_DEVTOOL_PROJECT=<project-name>export JOYGEN_DEVTOOL_PROJECT_PROJECTID=<project-id>export JOYGEN_DEVTOOL_PROJECT_TASKID=<task-id>export JOYGEN_DEVTOOL_USER_COOKIE=<cookie>export JOYGEN_DEVTOOL_USER_USERNAME=<erp>export JOYGEN_DEVTOOL_DEPLOY_PROVIDER=sandboxexport JOYGEN_DEVTOOL_DEPLOYSTAT_PROVIDER=sandboxexport JOYGEN_DEVTOOL_LOGQUERY_PROVIDER=sandboxexport JOYGEN_DEVTOOL_OUTPUT_FORMAT=jsonexport JOYGEN_DEVTOOL_DEPLOY_WAIT_TIMEOUT_MS=300000```  
  
说明：  
  
- `JOYGEN_DEVTOOL_PROJECT` 标识当前项目。  
- `JOYGEN_DEVTOOL_PROJECT_PROJECTID` 是沙盒平台项目 ID。  
- `JOYGEN_DEVTOOL_PROJECT_TASKID` 是沙盒平台任务 ID，deploy start 创建沙盒时会与 projectId 同级传给下游接口。  
- `JOYGEN_DEVTOOL_USER_COOKIE` 用于沙盒平台鉴权，不要在输出中展示。  
- `JOYGEN_DEVTOOL_USER_USERNAME` 用于记录用户身份。  
- 三个 provider 路由变量应设置为 `sandbox`，确保部署、状态和日志都走沙盒链路。  
  
## 推荐组合流程  
  
1. 完成代码修改。  
2. 执行部署并等待结果：  
  
```bash  
joygen-devtool deploy start --env dev --output json```  
  
3. 如果部署失败，拉取构建日志并分析原因：  
  
```bash  
joygen-devtool logs search --env dev --live build --limit 200 --output json```  
  
4. 如果部署成功，每隔 8 秒拉取运行日志：  
  
```bash  
joygen-devtool logs search --env dev --live running --limit 200 --output json```  
  
5. 如果运行日志显示启动失败，基于日志继续修改代码，再重新执行 `deploy start`。  
6. 如果运行日志显示启动成功，本次操作完成。  
  
## 结果判定  
  
可以判定沙盒验证通过的条件：  
  
- `deploy start` 返回结构化成功结果，并已等待到 ready。  
- 运行日志中出现应用启动完成、服务监听、ready、healthy 等启动成功信号。  
- 运行日志中没有启动失败、未捕获异常、端口绑定失败、依赖连接失败、配置缺失、进程退出等失败信号。  
  
不能判定通过的情况：  
  
- 只查询了旧沙盒状态，没有触发新部署。  
- `deploy start` 返回失败。  
- 构建日志或运行日志显示失败。  
- 运行日志为空且没有启动成功信号。  
  
## 输出给用户  
  
最终回复要包含：  
  
- 是否实际触发沙盒部署。  
- 部署结果摘要。  
- 构建日志或运行日志摘要。  
- 关键字段：`sandboxId`、`runId`、`serviceUrl`、`traceId`、`bCode`。  
  
## Guardrails  
  
- 优先使用 `--output json`，保留结构化字段。  
- `logs search` 在本流程中使用 `--live build --limit 200` 或 `--live running --limit 200`。  
- 不要输出 cookie、token、资源密码等敏感信息。  
- `daemon run` 是内部命令，除排障外不要人工调用。