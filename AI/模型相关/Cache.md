# KV缓存
* 示意图
![[Pasted image 20260511205105.png]]
KV指的是Transformer架构核心变量的KV，由于生成一个token的时候需要依赖输入序列的所有token，而原初Transformer是无状态的，因此添加KV缓存可大幅提升速度

* FTS指全文检索，类似ES的词汇到文档的倒排索引
* 向量检索通常和FTS构成双路检索方案，从字面和语义召回记忆