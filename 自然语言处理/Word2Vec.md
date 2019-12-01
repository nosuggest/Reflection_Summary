# 从隐藏层到输出的Softmax层的计算有哪些方法？
- 层次softmax
- 负采样

# 层次softmax流程？
- 构造Huffman Tree
- 最大化对数似然函数
    - ![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9adl5ex45j305g00qmwz.jpg)
        - 输入层：是上下文的词语的词向量
        - 投影层：对其求和，所谓求和，就是简单的向量加法
        - 输出层：输出最可能的word
    - 沿着哈夫曼树找到对应词，每一次节点选择就是一次logistics选择过程，连乘即为似然函数
- 对每层每个变量求偏导，参考sgd

# 负采样流程？
- 统计每个词出现对概率，丢弃词频过低对词
- 每次选择softmax的负样本的时候，从丢弃之后的词库里选择（选择是需要参考出现概率的）
- 负采样的核心思想是：利用负采样后的输出分布来模拟真实的输出分布

# word2vec两种方法各自的优势?
- **Mikolov 的原论文，Skip-gram 在处理少量数据时效果很好，可以很好地表示低频单词。而 CBOW 的学习速度更快，对高频单词有更好的表示**
- Skip-gram的时间复杂度是o(kv),CBOW的时间复杂度o(v)

# 怎么衡量学到的embedding的好坏?
- 从item2vec得到的词向量中随机抽出一部分进行人工判别可靠性。即人工判断各维度item与标签item的相关程度，判断是否合理，序列是否相关
- 对item2vec得到的词向量进行聚类或者可视化

# word2vec和glove区别？
- word2vec是基于邻近词共现，glove是基于全文共现
    - word2vec利用了负采样或者层次softmax加速，相对更快
    - glove用了全局共现矩阵，更占内存资源
- word2vec是“predictive”的模型，而GloVe是“count-based”的模型

# 你觉得word2vec有哪些问题？
- 没考虑词序
- 对于中文依赖分词结果的好坏
- 新生词无法友好处理
- 无正则化处理