# 我奶奶都能看懂的 AB 实验入门：模型篇

可能非常多的同学对于如何使平台工具进行 AB 实验驾轻就熟，但是却对平台背后工作的原理一知半解：为什么我需要创建参数？层究竟是怎样一种存在？为什么除了观察指标的升降以外还需要关注显著性和进行假设检验？可能这些问题并不妨碍你进行实验，但是如果拥有更完备的知识，你的实验会设计的更加合理，过程会进行的更加流畅，实验效率也得到提升

这不是一系列的安利文章，也不是面向工程师的技术文档，正如标题暗示的，它定位是具有科普性质的文章入门指南，我希望能做到通俗但又希望不失深度。标题当然是夸张了，阅读这篇文章仍然需要互联网行业的一些基础知识。所以我也直接省略了什么是 AB 实验以及 AB 实验场景等等内业内的普世知识，

## 概述

什么是实验模型？在 AB 测试的语境下，实验模式是一系列关于流量分配、样本选取以及指标统计等方法的集合。实验模型至关重要，它直接决定了工程上 AB 实验系统应该如何设计和编码，以及工程师或者产品经理能够多有效多便捷的进行 AB 实验。重中之重的是，如果我们的新功能上线依赖 AB 系统做出决策，那么可以说 AB 模型甚至决定了一个产品的生与死。

不同平台不同公司使用的模型各不相同，这里我们重点以 Google 论文 [Overlapping Experiment Infrastructure:
More, Better, Faster Experimentation](https://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/36500.pdf) 中描述的实验模型为蓝本，向大家阐述 AB 实验中涉及的概念，需要解决的问题，成熟的解决方案等等。虽然我们是以 Google 为例，但是这些内容在大部分实验模型中都是通用的。最后我们再简单看一看能找到公开资料中，其他公司的一些实验模型。

提前声明我并不会完全搬运论文中的内容，首先是没有必要，论文毕竟还是为科研人员和专业人士准备的，触及的内容越多所需要具备的专业知识也就越多，并且细枝末节的内容反而会干扰主线的阅读；再者个人水平有限，对论文依然没有十足的理解，所以只拾取了有把握的节选；最后相信读者中有不少就是涉猎这方面的专业人士，不足之处欢迎指出，敬请谅解。

AB 实验中还会涉及到概率方面的知识，例如如何进行假设检验，什么是显著性等等。这些内容我们放在下一篇“概率篇”中再详细描述

## “层 (Layer)”的由来

让我们先抛开互联网产品的业务场景，从一个生活中就能实现的 AB 实验开始。

假设我有十二盆相同生长状况的花朵，同时我想了解光照时长对花儿生长的影响，于是决定进行 AB 实验，方案如下：我将花了随机平均分为 AB 两组，每组六盆。A 组每天相同光照条件 5 小时，B 组每天与 A 组相同的光照条件 10 小时，同时保证两个实验组的其他生长条件（水分、供氧等）相同。一段时间后，比较两组花儿的生长状况（生长高度、重量等），即可判断出光照时长对于花儿生长状况的影响。

## 参考资料

- [Overlapping Experiment Infrastructure:
More, Better, Faster Experimentation](https://static.googleusercontent.com/media/research.google.com/zh-CN//pubs/archive/36500.pdf)
- [Overlapping Experiment Infrastructure:
More, Better, Faster](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/papers/Overlapping_Experiment_Infrastructure_More_Be.pdf)