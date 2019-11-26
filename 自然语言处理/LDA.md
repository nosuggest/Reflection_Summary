# 详述LDA原理？
- 从狄利克雷分布α中取样生成文档i的主题分布![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b8vd6yi8j300c00f0qn.jpg)
    - 多项式分布的共轭分布是狄利克雷分布
    - 二项式分布的共轭分布是Beta分布
- 从主题的多项式![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b8vd6yi8j300c00f0qn.jpg)分布中取样生成文档i第j个词的主题![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b8vq647gj300i00e0r3.jpg)
- 从狄利克雷分布β中取样生成主题![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b8vq647gj300i00e0r3.jpg)对应的词语分布![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b8u1wbj1j300q00j3y9.jpg)
- 从词语的多项式分布![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b8u1wbj1j300q00j3y9.jpg)中采样最终生成词语![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b8uydisuj300n00e0s0.jpg)
- 文档里某个单词出现的概率可以用公式表示：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b9y6avdtj306e01jdfo.jpg)
- 采用EM方法修正p(w/z)p(z/d)直至收敛

# LDA中的主题矩阵如何计算?
- 给矩阵wk和kj随机赋值，其中wk是每个主题中每个单词出现的次数，kj是每个文档中每个主题出现的次数，虽然这些次数还只是随机数，我们还是可以根据这些次数，利用Dirichlet分布计算出每个主题中每个单词最可能出现的概率，以及每个文档中每个主题最可能出现的概率
- 每个主题产生单词的概率：p(z/w,d) = p(w/z)p(z/d)
- 由于确定了这个单词是哪个主题产生的，相当于Dirichlet分布中a的值发生了改变，于是计算出新的词-主题的概率矩阵+主题-文档的概率矩阵
- 最后主题-文档的概率矩阵即为所求

# LDA的共轭分布解释下?
以多项式分布-狄利克雷分布为例，我们的多项式分布θ先验分布π(θ)，及加了样本信息x后的后验分布π(θ/x)都满足狄利克雷分布，则称狄利克雷分布为多项式分布的共轭分布


# PLSA和LDA的区别?
- LDA是加了狄利克雷先验的PLSA
    - PLSA的p(z/d)和p(w/z)都是直接EM估计的，而LDA都是通过狄利克雷给出的多项式参数下估计出来的
- LDA是贝叶斯思想，PLSA是MLE

# 怎么确定LDA的topic个数
- 对文档d属于哪个topic有多不确定，这个不确定程度就是Perplexity
- 多次尝试，调优perplexity-topic number曲线
    - 困惑度越小，越容易过拟合
    - 某个词属于某个主题的困惑度：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b7zjns8uj305i012jr7.jpg)，某个文章的困惑度即为词的连乘：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9b83z3d22j304q01dweb.jpg)

# LDA和Word2Vec区别？LDA和Doc2Vec区别？
- LDA比较是doc，word2vec是词
- LDA是生成的每篇文章对k个主题对概率分布，Word2Vec生成的是每个词的特征表示
    - LDA的文章之间的联系是主题，Word2Vec的词之间的联系是词本身的信息
- LDA依赖的是doc和word共现得到的结果，Word2Vec依赖的是文本上下文得到的结果
