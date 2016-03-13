---
layout: post
title: "如何完成比难更难的事？"
description: ""
category: articles
tags: [阅读]
---

![AlphaGo](http://7xq85r.com1.z0.glb.clouddn.com/836ad67d62458c744c2253b87f14ea51.jpeg)

昨天是Alpha Go对李世石的第三盘，在做饭间我看了开局，阿法狗又祭出了新招式，感觉李世石要0：5接受阿法狗教育的命运不可阻挡。虽然我也没有看过柯洁的棋，作为人类世界冠军，年少轻狂这种词也不适合戴他头上，唯有希望Google能够安排他们也战一次。

没有看棋，我转而去看了DeepMind发表在Nature上面的论文，仔细的学习了下阿法狗是怎么训练，在实战是怎么样运作的。由于关于机器学习我只有皮毛的知识，也没用实践过，我的讲解应该是不难理解的。（也有可能有细节上的错误）

##梗概##

阿法狗是由这么四部分组成的：

1. 走子策略网络（policy network）
2. 快速走子（fast rollout）
3. 局面评估网络（value network）
4. 蒙特卡洛树搜索（monte-carlo tree search，MCTS）

上面这三部分，在赛前中进行数据训练得出来的**功能**，而第4部分，则是在实战中，将1-3的功能来进行计算的部分。


