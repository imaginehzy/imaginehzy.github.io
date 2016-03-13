---
layout: post
title: "如何训练出一只阿法狗"
description: ""
category: articles
tags: [AI]
---



![AlphaGo](http://7xq85r.com1.z0.glb.clouddn.com/836ad67d62458c744c2253b87f14ea51.jpeg)

昨天是Alpha Go对李世石的第三盘，在做饭间我看了开局，阿法狗又祭出了新招式，感觉李世石要0：5接受阿法狗教育的命运不可阻挡。虽然我也没有看过柯洁的棋，作为人类世界冠军，年少轻狂这种词也不适合戴他头上，唯有希望Google能够安排他们也战一次。

没有看棋，我转而去看了DeepMind发表在Nature上面的论文**Mastering the Game of Go with Deep Neural Networks and Tree Search**，仔细的学习了下阿法狗是怎么训练，在实战是怎么样运作的。由于关于机器学习我只有皮毛的知识，也没用实践过，我的讲解应该是不难理解的。（也有可能有细节上的错误）

##梗概##

阿法狗是由这么四部分组成的：

1. 走子策略网络（policy network）
2. 快速走子（fast rollout）
3. 局面评估网络（value network）
4. 蒙特卡洛树搜索（monte-carlo tree search，MCTS）

上面这三部分，在赛前中进行数据训练得出来的**功能**，而第4部分，则是在实战中，将1-3的功能来进行计算的部分。先解释一下前三部分设置，蒙特卡洛树搜索留在最后部分讲。

**走子策略网络（policy network）** 负责给出在当前布局下，可以走的盘面选择的走子概率。对于己方，就是概率越高越推荐走的着法；对于敌方，就是猜测对方最后可能走子的敌方。**快速走子（fast rollout）** 和走子策略网络其实是做同样的功能，只不过它用的是传统的专家归纳的特征pattern和知识作为依据。这两方面的实现在后面一点讲。

**局面评估网络（value network）** 负责的是给出当前布局，己方是优势是劣势的判断。按照Facebook的阿法狗竞争项目DarkForest的负责人田渊栋的介绍，对于局面优劣的判断是由来已久围棋AI的难点，他们黑暗森林也没用做这个部分。以往的局面优劣判断，我估计就是点目+假设点棋子对周围有控制+形状厚薄判断之类的综合。而不是对局面以及未来走势完整的给出判断。在这次人机大战中，似乎AI的给出局面判断，在开始时是往往和常人感觉有点不一样的。

##深度学习##

这部分我只能将我知道的简单说明一下。首先我们举一个人脸识别的例子，这些在视觉识别上都差不多。

![视觉识别例子](http://7xq85r.com1.z0.glb.clouddn.com/cdd9ea4ba57d99d3c5db277fcec537ea.png)

![人脸识别](http://7xq85r.com1.z0.glb.clouddn.com/564ba0c236568.jpg)

假设是一张64*64像素的图，通常会将其二值化，就是变成黑白灰的灰度图。然后第一层提取特征，乃至二三四层，越来抽象。计算机是这样识别人脸的，据他们说这也是参照人类的神经网络结构而成的。具体的说法来源我没有考证过。

![神经网络](http://7xq85r.com1.z0.glb.clouddn.com/da03415312964e76cef62170685a87c8.png)







