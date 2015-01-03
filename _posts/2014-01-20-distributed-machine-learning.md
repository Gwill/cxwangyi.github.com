# 分布式机器学习的故事

## 初衷 ##

从毕业加入Google开始做分布式机器学习，到后来转战腾讯，负责广告业务的智能算法和策略，至今已经七年了。回顾自己所见所闻以及亲身实践，有一系列感触。其中有一些和目前如日中天却同时又方兴未艾的互联网经济大潮自然契合。比如有人问机器学习经过了数十年的发展，怎么也没有出现《终结者》里那样智商超越人类的“天网系统”。我想说，其实基于大数据的现代机器学习系统，已经初具“天网”的雏形。再比如有很多工程师讨论哪种并行计算架构适合机器学习。可是从我的经历看，这有点儿像是一个伪命题。因为虽然在验证一个新的并行算法的正确性的时候，我们有时候恰好可以利用现有的框架，尽量快速实现，但是任何一个有价值的机器学习方法，都值得拥有自己独特的架构。所以重点在有一个分布式操作系统，方便大家开发自己需要的架构（框架），来支持相应的算法。如果你关注大数据和互联网，那么欢迎听我说几个故事。也许你会有同样地感触。

因为我想说的内容有点儿多（毕竟总结了七年的经历），所以我分开成若干个章节，一章一个故事。

* 前言
   1. [大数据带来的新机遇](http://cxwangyi.github.io/story/00_0_new_era.md.html)
   1. [分布式机器学习的评价标准](http://cxwangyi.github.io/story/00_1_principles.md.html)
* 故事
   1. [pLSA和MPI：大数据的首要目标是“大”而不是“快”](http://cxwangyi.github.io/story/01_plsa_and_mpi.md.html)
   1. [LDA和MapReduce：“可扩展”的基础是数据并行](http://cxwangyi.github.io/story/02_lda_and_mapreduce.md.html)
   1. [Rephil和MapReduce：描述长尾数据的数学模型](http://cxwangyi.github.io/story/03_rephil_and_mapreduce.md.html)
   1. LDA和Pregel
   1. Clustering和Pregel
   1. 分类器和GBR
   1. SETI：Online pCTR
   1. MapReduce Lite
   1. Deep Learning和DistBelief
   1. Peacock
   1. [Docker：分布式系统的软件工程革命（上）](http://cxwangyi.github.io/story/docker_revolution_1.md.html)
