# 主要使用了什么机制?
Attention机制，针对不同的广告，用户历史行为与该广告的权重是不同的。

# activation unit的作用
基于Attention机制，结合当前需对比的不同目标，将用户历史行为进行不同的权重分配。
- ![](https://tva1.sinaimg.cn/large/006tNbRwgy1ga15ugjlkmj308901imx1.jpg)
- 通常用户兴趣可以由历史行为(点击/浏览/收藏)等合并得到，及![](https://tva1.sinaimg.cn/large/006tNbRwgy1ga15vxtcu8j302w01imwy.jpg)
- activation unit在这种思路上，认为面对不同的对象Va兴趣的权重Wi应该也是变换而不是固定的，所以用了g(ViVa)来动态刻画不同目标下的历史行为的不同重要性

# DICE怎么设计的
- 先对input数据进行bn，在进行sigmoid归一化到0-1，再进行一个加权平衡alpha*(1-x_p)`*`x+x_p`*`x
    - x_p=tf.sigmoid(tf.layers.batch_normalization(x, center=False, scale=False,training=True))
    - aplha*(1-x_p)*x+x_p*x

# DICE使用的过程中，有什么需要注意的地方
- 在用batch_normalization的时候，需要设置traning=True，否则在做test的时候，获取不到training过程中的各batch的期望
- test的时候，方差计算利用的是期望的无偏估计计算方法:E(u^2)`*`m/(m-1)