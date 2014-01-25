---
layout: post
title: "分布式机器学习的故事"
description: ""
category:
tags: []
---
{% include JB/setup %}


从毕业加入Google开始做分布式机器学习，到后来转战腾讯广告业务，至今已经七年了。我想说说我见到的故事和我自己的实践经历。这段经历给我的感觉是：虽然在验证一个新的并行算法的正确性的时候，我们可以利用现有框架，尽量快速实现，但是**任何一个有价值的机器学习思路，都值得拥有自己独特的架构。所以重点在有一个分布式操作系统，方便大家开发自己需要的架构（框架），来支持相应的算法**。如果你关注大数据，听完我说的故事，应该会有感触。

## 大数据和分布式机器学习

### 特点

说故事之前，先提纲挈领的描述一下我们要解决的问题的特点。我见过的有价值的大规模机器学习系统，基本都有三个特点：

1. **可扩展**。可扩展的意思是“投入更多的机器，能处理更大的数据”。而传统的并行计算要的是：“投入更多机器，数据大小不变，计算速度更快”。这是我认识中“大数据”和传统并行计算研究目标不同的地方。如果只是求速度快，那么multicore和GPU会比分布式机器学习的ROI更高。

   * 有一个框架（比如MPI或者MapReduce或者自己设计的），支持fault recovery。Fault recovery是可扩展的基础。现代机群系统都是很多用户公用的，其中任何一个进程都有可能被更高优先级的进程preempted。一个job涉及数千个进程（task processes），十分钟里一个进程都不挂的概率很小。而如果一个进程挂了，其他进程都得重启，那么整个计算任务可能永远都不能完成。

1. **数学模型要根据架构和数据做修改**。这里有两个原因：

    * 因为大数据基本都是长尾分布的，而papers里的模型基本都假设数据是指数分布的（想想用SVD做component analysis其实假设了Gaussian distributed，latent Dirichlet allocation假设了multimonial distribution。）。真正能处理大数据的数学模型，都需要能更好的描述长尾数据。否则，模型训练就是忽视长尾，而只关注从“大头”数据部分挖掘“主流”patterns了。

    * 很多机器学习算法（比如MCMC）都不适合并行化。所以往往需要根据模型的特点做一些算法的调整。有时候会是approximation。比如AD-LDA算法是一种并行Gibbs sampling算法，但是只针对LDA模型有效，对其他大部分模型都不收敛，甚至对LDA的很多改进模型也不收敛。

1. **引入更多机器的首要目的不是提升性能，而是能处理更大的数据**。用更多的机器，处理同样大小的数据，期待speedup提高——这是传统并行计算要解决的问题——是multicore、SMP、MPP、GPU还是Beowolf cluster上得分布式计算不重要。在大数据情况下，困难点在问题规模大，数据量大。此时，引入更多机器，是期待能处理更大数据，总时间消耗可以不变甚至慢一点。分布式计算把数据和计算都分不到多台机器上，在存储、I/O、通信和计算上都要消除瓶颈。

上述三个特点，会在实践中要求“一个有价值的算法值得也应该有自己独特的框架”。

### 概念

在开始说故事之前，先正名几个概念：MPI、MapReduce都是**框架（framework）**。MPICH2和Apache Hadoop分别是这两个框架的**实现（implementations）**。后面还会提到BSP框架，它的一个著名实现是Google Pregel。

MPI这个框架很灵活，对程序结构几乎没有太多约束，以至于大家有时把MPI称为一组**接口（API）**。

这里，MPICH2和Hadoop都是很大的系统——除了实现框架（允许程序员方便的编程），还实现了资源管理和分配，以及资源调度的功能。这些功能在Google的系统里是分布式操作系统负责的，而Google MapReduce和Pregel都是在分布式操作系统基础上开发的，框架本身的代码量少很多，并且逻辑清晰易于维护。当然Hadoop已经意识到这个问题，现在有了YARN操作系统。（YARN是一个仿照UC Berkeley AMPLab的Mesos做的系统。关于这个“模仿”，又有另一个故事。）

## 故事

### pLSA和MPI：大数据的首要目标是“大”而不是“快”

我2007年毕业后加入Google做研究。我们有一个同事叫张栋，他的工作涉及pLSA模型的并行化。这个课题很有价值，因为generalized matrix decomposition实际上是collaborative filtering的generalization，是用户行为分析和文本语义理解的共同基础。几年后的今天，我们都知道这是搜索、推荐和广告这三大互联网平台产品的基础。

当时的思路是用MPI来做并行化。张栋和宿华合作，开发一套基于MPI的并行pLSA系统。MPI是1980年代流行的并行框架，进入到很多大学的课程里，熟悉它的人很多。MPI这个框架提供了很多基本操作：除了点对点的Send, Recv，还有广播Bdcast，甚至还有计算加通信操作，比如AllReduce。

MPI很灵活，描述能力很强。因为MPI对代码结构几乎没有什么限制——任何进程之间可以在任何时候通信——所以很多人不称之为框架，而是称之为“接口”。

但是Google的并行计算环境上没有MPI。当时一位叫白宏杰的工程师将MPICH2移植到了Google的分布式操作系统上。具体的说，是重新实现MPI里的Send, Recv等函数，调用分布式操作系统里基于HTTP RPC的通信API。

MPI的AllReduce操作在很多机器学习系统的开发里都很有用。因为很多并行机器学习系统都是各个进程分别训练模型，然后再合适的时候（比如一个迭代结束的时候）大家对一下各自的结论，达成共识，然后继续迭代。这个“对一下结论，达成共识”的过程，往往可以通过AllReduce来完成。

如果我们关注一下MPI的研究，可以发现曾经有很多论文都在讨论如何高效实现AllReduce操作。比如我2008年的[博文](http://cxwangyi.wordpress.com/2008/12/04/the-highly-efficient-allreduce-in-mpich2/)里提到一种当时让我们都觉得很聪明的一种算法。这些长年累月的优化，让MPICH2这样的系统的执行效率（runtime efficiency）非常出色。

基于MPI框架开发的pLSA模型虽然效率高，并且可以处理相当大的数据，但是还是不能处理Google当年级别的数据。原因如上节《概念》中所述——MPI框架没有自动错误恢复功能，而且这个框架定义中提供的灵活性，让我们很难改进框架，使其具备错误恢复的能力。

具体的说，MPI允许进程之间在任何时刻互相通信。如果一个进程挂了，我们确实可以请分布式操作系统重启之。但是如果要让这个“新生”获取它“前世”的状态，我们就需要让它从初始状态开始执行，接收到其前世曾经收到的所有消息。这就要求所有给“前世”发过消息的进程都被重启。而这些进程都需要接收到他们的“前世”接收到过的所有消息。这种数据依赖的结果就是：所有进程都得重启，那么这个job就得重头做。

一个job哪怕只需要10分钟时间，但是这期间一个进程都不挂的概率很小。只要一个进程挂了，就得重启所有进程，那么这个job就永远也结束不了了。

虽然我们很难让MPI框架做到fault recovery，我们可否让基于MPI的pLSA系统支持fault recovery呢？原则上是可以的——最简易的做法是checkpointing——时不常的把有所进程接收到过的所有消息写入一个分布式文件系统（比如GFS）。或者更直接一点：进程状态和job状态写入GFS。Checkpointing是下文要说到的Pregel框架实现fault recovery的基础。

但是如果一个系统自己实现fault recovery，那还需要MPI做什么呢？做通信？——现代后台系统都用基于HTTP的RPC机制通信了，比如和Google的Stubby、Facebook的Thrift、腾讯的Poppy还有Go语言自带的rpc package。做进程管理？——在开源界没有分布式操作系统的那些年里有价值；可是今天（2013年），Google的Borg、AMPLab的Mesos和Yahoo!的YARN都比MPICH2做得更好，考虑更全面，效能更高。

### LDA和MapReduce：“可扩展”的基础——数据并行。

因为MPI在可扩展性上的限制，	我们可以大致理解为什么Google的并行计算架构上没有实现经典的MPI。同时，我们自然的考虑Google里当时最有名的并行计算框架MapReduce。

MapReduce的风格和MPI截然相反。MapReduce对程序的结构有严格的约束——计算过程必须能在两个函数中描述：map和reduce；输入和输出数据都必须是一个一个的records；任务之间不能通信，整个计算过程中唯一的通信机会是map phase和reduce phase之间的shuffuling phase，这是在框架控制下的，而不是应用代码控制的。

pLSA模型的作者Thomas Hoffmann提出的机器学习算法是EM。EM是各种机器学习inference算法中少数适合用MapReduce框架描述的——map phase用来推测（inference）隐含变量的分布（distributions of hidden variables），也就是实现E-step；reduce phase利用上述结果来更新模型，也即是M-step。

但是2008年的时候，pLSA已经被新兴的LDA掩盖了。LDA是pLSA的generalization：一方面LDA的hyperparameter设为特定值的时候，就specialize成pLSA了。从工程应用价值的角度看，这个数学方法的generalization，允许我们用一个训练好的模型解释任何一段文本中的语义。而pLSA只能理解训练文本中的语义。（虽然也有ad hoc的方法让pLSA理解新文本的语义，但是大都效率低，并且并不符合pLSA的数学定义。）这就让继续研究pLSA价值不明显了。

另一方面，LDA不能用EM学习了，而需要用更generalized inference算法。学界验证效果最佳的是Gibbs sampling。作为一种Markov Chain Monte Carlo（MCMC）算法，顾名思义，Gibbs sampling是一个顺序过程，按照定义不能被并行化。

但是2007年的时候，UC Irvine的David Newman团队发现，对于LDA这个特定的模型，Gibbs sampling可以被并行化。具体的说，把训练数据拆分成多份，用每一份独立的训练模型。每隔几个Gibbs sampling迭代，这几个局部模型之间做一次同步，得到一个全局模型，并且用这个全局模型替换各个局部模型。这个研究发表在NIPS上，题目是：Distributed Inference for Latent Dirichlet Allocation。

上述做法，在2012年Jeff Dean关于distributed deep leearning的[论文](http://research.google.com/archive/large_deep_networks_nips2012.html)中，被称为**data parallelism**（数据并行）。如果一个算法可以做数据并行，很可能就是可扩展（scalable）的了。

David Newman团队的发现允许我们用多个map tasks并行的做Gibbs sampling，然后在reduce phase中作模型的同步。这样，一个训练过程可以表述成一串MapReduce jobs。我用了一周时间在Google MapReduce框架上实现实现和验证了这个方法。后来在同事Matthew Stanton的帮助下，优化代码，提升效率。但是，因为每次启动一个MapReduce job，系统都需要重新安排进程（re-schedule）；并且每个job都需要访问GFS，效率不高。在当年的Google MapReduce系统中，1/3的时间花在这些杂碎问题上了。后来实习生司宪策在Hadoop上也实现了这个方法。我印象里Hadoop环境下，杂碎事务消耗的时间比例更大。

随后白红杰在我们的代码基础上修改了数据结构，使其更适合MPI的AllReduce操作。这样就得到了一个高效率的LDA实现。我们把用MapReduce和MPI实现的LDA的Gibbs sampling算法发表在[这篇论文](http://plda.googlecode.com/files/aaim.pdf)里了。

当我们踌躇于MPI的扩展性不理想而MapReduce的效率不理想时，Google MapReduce团队的几个人分出去，开发了一个新的并行框架Pregel。当时Pregel项目的tech lead访问中国。这个叫Grzegorz Malewicz的波兰人说服了我尝试在Pregel框架下验证LDA。但是在说这个故事之前，我们先看看Google Rephil——另一个基于MapReduce实现的并行隐含语义分析系统。

### Rephil和MapReduce：描述长尾数据的数学模型

Google Rephil是Google AdSense背后广告相关性计算的头号秘密武器。但是这个系统没有发表过论文。只是其作者（博士Uri Lerner和工程师Mike Yar）在2002年在湾区举办的几次小规模交流中简要介绍过。所以Kevin Murphy把这些内容写进了他的书《Machine Learning: a Probabilitic Perspecitve》里。在吴军博士的《数学之美》里也提到了Rephil。

Rephil的模型是一个全新的模型，更像一个神经元网络。这个网络的学习过程从Web scale的文本数据中归纳海量的**语义**——比如“apple”这个词有多个意思：一个公司的名字、一种水果、以及其他。当一个网页里包含"apple", "stock", "ipad"等词汇的时候，Rephil可以告诉我们这个网页是关于apple这个公司的，而不是水果。

这个功能按说pLSA和LDA也都能实现。为什么需要一个全新的模型呢？

从2007年至今，国内外很多团队都尝试过并行化pLSA和LDA。心灵手巧的工程师们，成功的开发出能学习数万甚至上十万语义（latent topics）的训练系统。但是不管大家用什么训练数据，都会发现，得到的大部分语义（相关的词的聚类）都是非常类似，或者说“重复”的。如果做一个“去重”处理，几万甚至十万的语义，就只剩下几百几千了。

这是怎么回事？

如果大家尝试着把训练语料中的低频词去掉，会发现训练得到的语义和用全量数据训练得到的差不多。换句话说，pLSA和LDA模型的训练算法*没有在意低频数据*。

为什么会这样呢？因为pLSA和LDA这类概率模型的主要构造单元都是*指数分布*（exponential distributions）。比如pLSA假设一个文档中的语义的分布是multinomial的，每个语义中的词的分布也是multinomial的。因为multinomial是一种典型的指数分布，这样整个模型描述的海量数据的分布，不管哪个维度上的marginalization，都是指数分布。在LDA中也类似——因为LDA假设各个文档中的语义分布的multinomial distributions的参数是符合Dirichlet分布的，并且各个语义中的词的分布的multinomial distributions的参数也是符合Dirichlet分布的，这样整个模型是假设数据是指数分布的。

可是Internet上的实际数据基本都不是指数分布的——而是长尾分布的。至于为什么是这样？可以参见2006年纽约时报排名畅销书The Long Tail: Why the Future of Business is Selling Less of More。或者看看其作者Chris Anderson的博客[The Long Tail](http://www.thelongtail.com)。

长尾分布的形状大致如下图所示：

<img src="http://www.thelongtail.com/conceptual.jpg"/>

其中x轴表示数据的类型，y轴是各种类型的频率，少数类型的频率很高（称为*大头*，图中红色部分），大部分很低，但是大于0（称为*长尾*，图中黄色部分）。一个典型的例子是文章中词的分布，有个具体的名字Zipf's law，就是典型的长尾分布。而指数分布基本就只有大头部分——换句话说，如果我们假设长尾数据是指数分布的，我们实际上就把尾巴给割掉了。

割掉数据的尾巴——这就是pLSA和LDA这样的模型做的——那条长尾巴覆盖的多种多样的数据类型，就是Internet上的人生百态。理解这样的百态是很重要的。比如百度和Google为什么能如此赚钱？因为互联网广告收益。传统广告行业，只有有钱的大企业才有财力联系广告代理公司，一帮西装革履的高富帅聚在一起讨论，竞争电视或者纸媒体上的广告机会。互联网广告里，任何人都可以登录到一个网站上去投放广告，即使每日广告预算只有几十块人民币。这样一来，刘备这样织席贩屡的小业主，也能推销自己做的席子和鞋子。而搜索引擎用户的兴趣也是百花齐放的——从人人爱戴的陈老师苍老师到各种小众需求包括“红酒木瓜汤”（一种丰胸秘方，应该出丰胸广告）或者“苹果大尺度”（在搜索范冰冰主演的《苹果》电影呢）。把各种需求和各种广告通过智能技术匹配起来，就酝酿了互联网广告的革命性力量。这其中，理解各种小众需求、长尾意图就非常重要了。

实际上，Rephil就是这样一个能理解百态的模型。因为它把Google AdSense的盈利能力大幅提升，最终达到Google收入的一半。两位作者荣获Google的多次大奖，包括Founders' Award。

而切掉长尾是一个很糟糕的做法。大家还记得小说《1984》里有这样一个情节吗？老大哥要求发布“新话”——一种新的语言，删掉自然英语中大部分词汇，只留下那些主流的词汇。看看小说里的人们生活的世界，让人浑身发毛，咱们就能体会“割尾巴”的恶果了。没有看过《1984》的朋友可以想象一下[水木](http://www.newsmth.net)首页上只有“全站十大”，连“分类十大”都删掉之后的样子。

既然如此，为什么这类模型还要假设数据是指数分布的呢？——实在是不得已。指数分布是一种数值计算上非常方便的数学元素。拿LDA来说，它利用了Dirichlet和multinomial两种分布的共轭性，使得其计算过程中，模型的参数都被积分给积掉了（integrated out）。这是AD-LDA这样的ad hoc并行算法——在其他模型上都不好使的做法——在LDA上好用的原因之一。换句话说，这是*为了计算方便，掩耳盗铃地假设数据是指数分布的*。

实际上，这种掩耳盗铃在机器学习领域很普遍。比如有个兄弟听了上面的故事后说：“那我们就别用概率模型做语义分析了，咱们还用矩阵分解吧？SVD分解怎么样？” 很不好意思的，当我们把SVD分解用在语义分析（称为LSA，latent semantic analysis）上的时候，我们还是引入了指数分布假设——Gaussian assumption或者叫normality assumption。这怎么可能呢？SVD不就是个矩阵分解方法吗？确实传统SVD没有对数据分布的假设，但是当我们用EM之类的算法解决存在**missing data**的问题——比如LSA，还有推荐系统里的协同过滤（collaborative filtering）——这时不仅引入了Gaussian assumption，而且引入了linearity assumption。当我们用其他很多矩阵分解方法做，都存在同样的
问题。

掩耳盗铃的做法怎么能存在得如此自然呢？这是因为Gaussian assumption有过很多成功的应用，包括通信、数据压缩、制导系统等。这些应用里，我们关注的就是数据中的低频部分；而高频部分（或者说距离mean比较远的数据）即使丢掉了，电话里的声音也能听懂，压缩还原的图像也看得明白，导弹也还是能沿着“最可能”靠谱的路线飞行。我们当然会假设数据是指数分布的，这样不仅省计算开销，而且自然的忽略高频数据，我们还鄙夷地称之为outlier或者noise。

可是在互联网的世界里，正是这些五花八门的outliers和noise，蕴含了世间百态，让数据不可压缩，从而产生了“大数据”这么个概念。处理好大数据的公司，赚得盆满钵满，塑造了一个个传奇。这里有一个听起来比较极端的说法**大数据里无噪声**——很多一开始频率很低，相当长尾，会被词过滤系统认为是拼写错误的queries，都能后来居上成为主流。比如“神马”，“酱紫”。

【这一节很长，却还没有进入正题——Rephil如何model长尾数据的？】

### LDA和Pregel

### Clustering和Pregel

### 分类器和GBR

### SETI：Online pCTR

### MapReduce Lite

### Deep Learning和DistBelief
### Peacock
