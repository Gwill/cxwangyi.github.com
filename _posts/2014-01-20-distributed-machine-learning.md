# “分布式机器学习的故事”系列分享

【更新 2015-03-01】在LinkedIn的同事王冠和朱平的协助下，在湾区的分布式机器学习系列分享结束了。感谢LinkedIn Events团队提供场地、器材和其他支持。一起参与的朋友们组成了一个微信群，继续保持沟通和交流。

## 内容

1. A New Era [slides](http://cxwangyi.github.io/distributed-machine-learning/00-a-new-era.pdf) [video](https://www.youtube.com/watch?v=dqE50moCwno)
1. Infrequent itemset mining [slides](http://cxwangyi.github.io/distributed-machine-learning/01-pfp.pdf) [video](https://www.youtube.com/watch?v=eCrhaUCqiiE)
1. Application Driven [slides](http://cxwangyi.github.io/distributed-machine-learning/02-app-driven.pdf) [video](https://www.youtube.com/watch?v=C7k0YSLAyX4)
1. Implement Your MapReduce [slides](http://cxwangyi.github.io/distributed-machine-learning/03-mapreduce.pdf) [video](https://www.youtube.com/watch?v=kZuJA5B2FtU)
1. Deep Learning [slides](http://cxwangyi.github.io/distributed-machine-learning/04-deeplearning.pdf) [video](https://www.youtube.com/watch?v=rqAu07z23yA)
1. Peacock and Latent Topic Modeling [slides](http://cxwangyi.github.io/distributed-machine-learning/05-peacock.pdf) [video](https://www.youtube.com/watch?v=IKGkQy_1xcw)

## 总结

* 互联网服务超越人工服务
* 集体智能超越人工智能
* 大数据是行为数据
* 大数据必然长尾
* 长尾数据无噪声
* 追求“大”比追求“快”重要
* 开发框架、而不是套用框架
* 工程技法和数学同样重要
* 远离 Java、远离 Python
* 有所谓好的系统，无所谓好的算法

## 初衷

从2007年博士毕业加入Google做机器学习至今已七年了，一直在工业界机器学习一线工作。尤其是从2010年开始担任腾讯广告的技术总监之后，一边组建团队，一边背负业务指标压力时，针对业务和产品设计开发机器学习技术。

在 Google 的工作让我有机会和同事们在 collaborative filtering、spectral clustering、frequent itemset mining、graph clustering、latent topic modeling等几个重要的研究方面做了一些尝试。基于其他同事在计算架构上的创新，我们在其中每个方面都有将文献中的数据处理能力提升1000倍的作品。这段经历让我能更好地针对问题选择方法，对我在腾讯的工作有很大帮助。在腾讯的工作集中在 retrieval system 和 ranking system，以及为了做好它们需要的机器学习技术。其间我们用 Go 语言开发的 Peacock至今是业界最大规模的 latent topic modeling system，在腾讯的广告、推荐和其他业务上使用。为 ranking 做的点击率预估系统也让我们团队成为 KDD Cup 2012的出题者和裁判团队。和学界的交流，收获和感触都很多。

这七年里的亲身参与和有幸旁观，让我总结了一些经验和形成了一些观点。有趣的是，这些观点与开源社区以及学术界对“大数据学习”的认识南辕北辙。2014年来到湾区工作之后，Linkedin的同事们鼓励和帮助我分享经历和经验。卡耐基梅隆大学的邢波（Eric Xing）教授也希望我给机器学习系的同学们做一个系列讲座。电子工业和人民邮电出版社的编辑朋友们也希望我完善和出版我的系列博客《分布式机器学习的故事》。

承蒙大家的鼓励和帮助，我们准备在湾区和匹兹堡同时开始一个系列的分享：第一次是分享我的经验总结和观点，后面十次每次分享一个我亲身经历过的工业界的实战故事。我们希望通过帮助朋友们模拟业界实战，营造一个深入思考和交流的机会。更清晰地判断大数据学习技术和业务生态发展方向。
