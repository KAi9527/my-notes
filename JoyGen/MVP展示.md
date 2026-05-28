# 提示词
请在当前项目中实现“按更新时间查询应用列表”的前后端联动功能。 要求： 1. 前端在应用列表页顶部搜索栏中增加“更新时间范围”筛选项，使用日期范围选择器。 2. 点击查询或筛选条件变化时，通过 services 调用真实后端接口 `/api/auth/applications/query`，请求体增加 `updateTimeStart`、`updateTimeEnd`。 3. 后端 `ApplicationQueryParam` 增加 `updateTimeStart`、`updateTimeEnd` 字段。 4. 按现有链路扩展 Controller -> ApplicationAppService -> ApplicationDomainService -> ApplicationGateway -> ApplicationGatewayImpl。 5. 在 `ApplicationGatewayImpl` 中使用 MyBatis-Plus `LambdaQueryWrapper` 对 `ApplicationDO.updateTime` 增加数据库层过滤： - `updateTime >= updateTimeStart` - `updateTime <= updateTimeEnd` 6. 更新应用时请确保 `updateTime` 会刷新为当前时间，保证更新时间筛选语义正确。 7. 不使用 Mock，不修改 package.json scripts、tsconfig、ESLint 配置和 Docker 相关文件。 8. 完成后请验证前端请求能通过 `/api` 代理打到 Java 后端，并说明涉及的文件和验证结果。

# 查询入口日志
ApplicationController.list query applications, param: