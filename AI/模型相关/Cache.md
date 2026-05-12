# KV缓存
* 示意图
![[Pasted image 20260511205105.png]]
KV指的是Transformer架构核心变量的KV，由于生成一个token的时候需要依赖输入序列的所有token，而原初Transformer是无状态的，因此添加KV缓存可大幅提升速度