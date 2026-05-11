# 定时任务
* 基于开源xxl-job上进行二次开发，单独部署为dts应用
* joy-gen和dts之间使用sdk进行通信
* 定时任务对于核心agent而言应当是一个tool，可使用这个tool进行定时任务新建、修改，并且和某个沙盒进行绑定
* dts调度执行器时，joy-gen应当提供入口方法，可根据参数找到对应沙盒的ip、方法路径进行执行

# Q
* api到cli是否类似为相应的api提供bash资源？

* 为什么更新沙盒状态的时候需要更新appId，应用会发生变化吗？DevCloudSandboxScheduleService 424