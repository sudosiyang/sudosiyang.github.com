---
layout: post
title: 程序员如何提一个好问题？
category : 前端技术
tags : 前端 javascript 译文
---

提出好的问题是在编写软件时的一个非常重要的技能。这么多年来我对此也算略有小成。这里有一些我用着觉得很棒的指导方针！

**开始**

我实际上是那种总是会问出愚蠢问题或“不好”问题的大信徒。我一直在问人们一些愚蠢并且完全可以通过谷歌搜索或搜索代码库解决的问题。大多数时候我都不愿意自己去搜索解决，但有的时候我又会无论如何都自己去搞定，而且也不会认为这如同世界末日一样可怕。

所以本文中列举的各个策略不是关于“在提问之前你必须要做的所有事情”，而是“一些可以帮助提出更好的问题并得到我想要的答案的要点！”。

![](https://jvns.ca/images/questions.png)
**何为好问题？**

我们的目标是提出易于回答的关于技术概念方面的问题。我时常碰到知识渊博并且这些知识也是我想知道的人，但他们并不总是知道如何确切地用最佳的方式解释。

如果有一系列好的问题，那么就可以帮助解答的人将他们所知道的内容有效地解释给我听，并指导他们告诉我我感兴趣的东西。那么我们该如何做到这一点呢？

**说明你所知道的**

这是我最喜欢的提问技巧之一！提问形式基本上是这样的：

* 说明到目前为止你对这个话题的理解
* 问“对吗？”

例如，我最近在和人（一个优秀的问题提问者）谈论网络！他们说“所以，我在这里的理解是有某个递归式dns服务器链……”。那是不正确的！实际上没有递归式DNS服务器链。（当你谈到递归式DNS服务器时，只涉及一个递归式服务器）因此他们说出他们当前的理解，可以方便我们澄清它实际上的工作原理。

我对rkt很感兴趣，但我不明白为什么rkt在运行容器时会比Docker占用更多的磁盘空间。

虽然“为什么rkt比Docker要使用更多的磁盘空间”不怎么像是正确的问题——我差不多知道代码是如何工作的，但我不明白为什么他们那样写代码。所以我把这个问题写到 rkt-dev 邮件列表：为什么rkt存储容器图像时不同于Docker？

我：

* 写下了我对rkt和Docker如何在磁盘上存储容器的理解
* 想出了几个我认为他们可能会按照他们的方式设计的原因
* 问“我的理解对吗？”

我得到的答案超级超级有帮助，正是我所寻找的。我花了很长时间以一种我满意的方式制定了这个问题，我很高兴我花了时间，因为它使我更好地明白了个中奥妙。

阐明你的理解并不容易（需要时间思考你所知道的并澄清你的想法），但效果会很好，更方便你要求帮助的人针对性地提出解答。

**问答案是事实的问题**

我有很多问题一开始有点模糊，如“SQL中的连接查询JOIN如何工作？”。这个问题不是很棒，因为连接查询如何工作有很多不同的部分！那么对方怎么知道我有兴趣学习的是什么？

我喜欢问那种答案是一个直截了当的事实的问题。例如，在SQL连接查询示例中，一些事实问题的答案可以是：

* 连接两个大小为N和M的表的时间复杂度是多少？是O(NM)吗？还是 O(NlogN)+ O(MlogM)？
* MySQL在进行连接查询之前是否始终将联结列排序作为第一步？
我知道Hadoop有时会“hash连接”——这是其他数据库引擎也使用的一个连接策略吗？
* 当我在一个索引列和一个未索引列之间进行连接时，我需要对非索引列进行排序吗？

当我问像这样超级具体的问题时，被问的人并不总是知道答案，但至少他们理解了我感兴趣的问题是怎么样的——很明显，我并不想知道如何使用连接查询，我就是想了解一些实现细节和算法。

**真诚地说出你不明白的地方**

很多时候当有人向我解释某事时，他们会说一些我不明白的东西。例如，可能有人正在向我解释一些关于数据库的东西，并说“好的，我们使用MySQL的乐观锁，因此……”。等等，我不知道什么是“乐观锁”啊。所以这需要提问了！ :  )

阻止某人接着说下去并提问“嘿，那是什么意思？”是一个超级重要的技能。我认为它是自信的工程师的属性之一，并且培养起来会大有裨益。我看到很多高级工程师经常要求澄清说明他或她不明白的地方——我觉得当你对自己的技能更有信心时，这更容易。

越是这么去做，在我要求别人澄清的时候就越是感觉自然。事实上，如果有人在我解释的时候不要求我澄清，我反而会担心他们不是真的有在听！

这也为问题回答者创造了在触及他们知识领域范围之外时可以承认的余地！很多时候，当我问某人问题时，如果问到他们不知道的东西。我问的人通常真的非常善于说“不，我不知道！”

**识别你不明白的术语**

当我开始当前这份工作时，我首先去了数据团队。当我看我的新工作需要什么的时候，有这些要求！Hadoop，Scalding，Hive，Impala，HDFS，zoolander，以及等等。我可能之前听说过Hadoop，但这些单词是什么意思我基本上是两眼一抹黑。其中一些是内部项目，其中一些是开源项目。所以我从要求帮助我理解每个术语的含义和它们之间的关系开始。我可能会问的一些问题是：

* HDFS是数据库吗？（不，它是一个分布式文件系统）
* Scalding使用Hadoop吗？（是）
* Hive使用Scalding吗？（不）

实际上我编写了一部关于所有术语的“字典”，因为术语实在太多，并且理解所有的术语意味着真正帮助我定位自己，以便于以后提出更好的问题。

**做一些研究**

在我键入上面的SQL问题时，我在Google搜索框中输入了“如何实现SQL连接”。我点击了一些链接，看到“哦，我知道了，有时有排序，有时有哈希连接，以前我听说过”这些话，然后写一些我遇到的更具体的问题。首先稍微Google一下，这可以帮助我写出更好的问题！

也就是说，我认为人们有时对“在没有谷歌搜索之前就不要提问题”这一原则太过苛刻——有时我在和某人一起吃午饭的时候，因为对他们的工作好奇，于是我就会问到相关的基本问题。这完全正常！

但是做研究非常有用，并且做足够的研究以便于提出一系列超赞的问题真的很有意思。

**决定去问谁**

在这里我主要谈论向你的同事问问题，因为大多数时候我都是向他们求助的。

询问同事时，我会思考到的一些问题是：

* 是提问的好时机吗？（如果他们在忙着做一件紧迫的事情，那么可能就不是好时机）
* 询问他们解答问题所需要的时间是不是可以节约我尽可能多的时间？（如果我问了一个问题，将花费他们5分钟回答，却将节约我2个小时的时间，那就棒棒哒 ： D）
* 他们需要多少时间来回答我的问题？如果我有半小时的问题要问，那么我可能会之后再安排一段时间，如果我只有一个快速的问题，那么我很有可能现在就问了。
* 这个人对这个问题而言是否过于太高级了？我认为这是很容易陷入的陷阱，那就是每个问题都去问最有经验/最有知识的人，而且每个问题的主题还各不相同。但实际上更好的办法是找一个知识稍微没那么渊博的人——通常他们可以回答大部分的问题，扩散负荷，而且他们还可以展示他们的知识（哈哈）。

我不总能做好这些事情，但考虑这些确实于我有所帮助。

此外，我通常会更多地去问更靠近问题的人——几乎每天我都会与之谈论的人，一般说来我更很倾向于去问他们问题，因为他们更了解我的工作背景，从而给我一个有用的答案。

《How to ask questions the smart way by ESR》是一个流行和相当有敌意的文档（它的开头陈述很烂，如‘我们称呼这样的人为“失败者”’）。内容关于如何在互联网上向陌生人提问。向互联网上的陌生人问问题是一个超级有用的技能，可以让你获取真正有用的信息，但这也是一类“硬模式”的问题。因为与你对话的人对你的情况知之甚微，所以更仔细地陈述你确切想要知道什么更佳。我不喜欢ESR文档，但它确实说明了一些有用的东西。文章的“How To Answer Questions in a Helpful Way”部分还是挺不错的。

**提出问题以显示不明显的内容**

更高级的问题提问形式是提出问题以揭示隐藏的假设或知识。这种问题实际上有两个目的——第一，得到答案（可能这个人知道但其他人不知道的信息），但也要指出，这里有一些隐藏的信息，并且共享这些信息是有用的。

Etsy的“Debriefing Facilitation Guide”中的“The Art of Asking Questions”部分就是在讨论已发生事件的背景下的一个非常好的入门介绍。以下是从该指南摘录的几个问题：

“当你怀疑这种类型的失败发生时，你想要寻找什么？”
“你怎么判定这种情况是‘正常’的？”
你是怎么知道数据库崩溃的？
你怎么知道那是你需要page的团队？
这些类似的问题（看起来很基本，但实际上并不明显）在某些权威人士提问的时候特别强大。我特别愿意看到经理/高级工程师问及这类基本但重要的问题，如“你是怎么知道数据库崩溃的？”，因为它为水平较低的人创造了以后提问相同问题的空间。

**回答问题**

André Arko的“How to Contribute to Open Source”里面有部分是我非常欣赏的

>既然你已阅读了 所有要点并pull请求，那么就开始查看你可以回答的问题。如果问题你以前就回答过，或者你刚刚阅读的文档就可以解答，那么用不了多少时间你就能发现这一点。回答你知道怎么回答的问题。

如果你正在攀登一个新项目，那么回答那些正在学习你刚学完的那些内容的人的问题，可谓是巩固知识的好方法。每当我第一次回答关于一个新主题的问题时，我总是会有一种“OMG，要是我答错了该怎么办啊，OMG”的感觉。但通常我都可以正确回答他们的问题，然后我就会感觉自己棒棒哒，好像自己更好地理解了主题！

**问题也是巨大的贡献**

好的问题可以为社区做出巨大的贡献！我回答了一些关于CDN的问题，并在 CDNs aren’t just for caching写出了答案。很多人告诉我，他们真的很喜欢这篇博文，我认为我问的这些问题帮助了很多人，不仅仅惠及自己。

很多人表示自己很喜欢回答问题！我认为将好的问题当作一件你可以做的超棒的事情，并放到对话中是很重要的，而不要只是认为“问好的问题，这样人们才只会稍微恼火，而不会非常非常恼火”。

> 翻译自：[https://jvns.ca/blog/good-questions/](https://jvns.ca/blog/good-questions/)