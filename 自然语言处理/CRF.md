# 阐述CRF原理？
- 首先X,Y是随机变量，P(Y/X)是给定X条件下Y的条件概率分布
- 如果Y满足马尔可夫满足马尔科夫性，及不相邻则条件独立
- 则条件概率分布P(Y|X)为条件随机场CRF

# 线性链条件随机场的公式是？
- ![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9ah66y1nxj30cr015747.jpg)

# CRF与HMM区别?
- CRF是判别模型求的是p(Y/X),HMM是生成模型求的是P(X,Y)
- CRF是无向图，HMM是有向图
- CRF全局最优输出节点的条件概率，HMM对转移概率和表现概率直接建模，统计共现概率

# Bert+crf中的各部分作用详解？
- Bert把中文文本进行了embedding，得到每个字的表征向量
- dense操作得到了每个文本文本对应的未归一化的tag概率
- CRF在选择每个词的tag的过程其实就是一个最优Tag路径的选择过程
    - CRF层能从训练数据中获得约束性的规则
        - 比如开始都是以xxx-B，中间都是以xxx-I，结尾都是以xxx-E
        - 比如在只有label1-I，label2-I..的情况下，不会出现label1-B