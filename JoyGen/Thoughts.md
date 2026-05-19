# 定时任务
* 基于开源xxl-job上进行二次开发，单独部署为dts应用
* joy-gen和dts之间使用sdk进行通信
* 定时任务对于核心agent而言应当是一个tool，可使用这个tool进行定时任务新建、修改，并且和某个沙盒进行绑定
* dts调度执行器时，joy-gen应当提供入口方法，可根据参数找到对应沙盒的ip、方法路径进行执行

# 后端
* agent通过cli自循环纠错的过程中如何加入人工介入途径？
* 20260512:当前cli中local/api provider均为mock实现，本期目标应当是完善实现，并结合记忆反馈agent

# 存量应用开发
* 当前选择了项目之后，HomeStore.templateProject会被赋值，导致创建项目的时候走默认创建逻辑；如要优化需要了解HomeStore.templateProject原来的作用
* CloudPreview/utils.ts (line 49)中硬编码了沙盒预览白名单，新增的项目资源类型需要改代码才能启动沙盒预览，后续需要优化

# Q
* api到cli是否类似为相应的api提供bash资源？

* 为什么更新沙盒状态的时候需要更新appId，应用会发生变化吗？DevCloudSandboxScheduleService 424

* 如何配置项目维度的项目资源？
* 如何往项目根目录中添加.mvn .joygen等配置文档？