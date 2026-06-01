![[Pasted image 20260601100622.png]]

# CLI优化
## deploy start命令修改
1. deploy start目前需要等待沙盒部署成功/失败之后返回部署结果，部署结果通过沙盒状态判断。方法：SandboxAtomicProvider.queryDeployStatusAtomic，该方法出参ready为true表示部署成功，failed为true表示失败，其他情况继续等待。等待超时时间默认5分钟，可以通过环境变量设置。环境变量在CLIContext处添加。
2. log search添加选项。--live选项表示实时查询接口日志，直接调用底层接口，逻辑参考SandboxDaemonRunner.run；在--live选项生效的情况下再使用--build和--running区分是单独查询构建日志或单独查询运行日志，只有--live的时候默认两种日志都返回；