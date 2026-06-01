![[Pasted image 20260601100622.png]]

# CLI优化
## deploy start命令修改
1. deploy start目前需要等待沙盒部署成功/失败之后返回部署结果，部署结果通过沙盒状态判断。deploy start需要同步方法：SandboxAtomicProvider.queryDeployStatusAtomic
2. 