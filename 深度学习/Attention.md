# Attention对比RNN和CNN，分别有哪点你觉得的优势？
- 对比RNN的是，RNN是基于马尔可夫决策过程，决策链路太短，且单向
- 对比CNN的是，CNN基于的是窗口式捕捉，没有受限于窗口大小，局部信息获取，且无序

# 写出Attention的公式？
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g986pbyaj0j308t019t8l.jpg)

# 解释你怎么理解Attention的公式的？
- Q:![](https://tva1.sinaimg.cn/large/006y8mN6ly1g986rjgs7qj301400g741.jpg),K:![](https://tva1.sinaimg.cn/large/006y8mN6ly1g986rz9d1ej301a00g741.jpg),V:![](https://tva1.sinaimg.cn/large/006y8mN6ly1g986shjz4tj301900g741.jpg)
    - 首先，我们可以理解为Attention把input重新进行了一轮编码，获得一个新的序列
- 除以![](https://tva1.sinaimg.cn/large/006y8mN6ly1g98734eqn7j300y00na9t.jpg)的目的是为了平衡qk的值，避免softmax之后过小
    - qk除了点击还可以直接拼接再内接一个参数变量等等
- Multi-Attention只是重复了h次的Attention，最后把结果进行拼接

# Attention模型怎么避免词袋模型的顺序问题的困境的？
增加了position Embedding
- 可以直接随机初始化
- 也可以参考Google的sin/cos位置初始化方法
    - 如此选取的原因之一是sin(a+b)=sin(a)cos(b)+cos(a)sin(b)。这很好的保证了位置p+k可以表示成p的线性变换，相对位置可解释

# Attention机制，里面的q,k,v分别代表什么？
- Q：指的是query，相当于decoder的内容
- K：指的是key，相当于encoder的内容
- V：指的是value，相当于encoder的内容

q和k对齐了解码端和编码端的信息相似度，相似度的值进行归一化后会生成对齐概率值（注意力值）。V对应的是encoder的内容，刚说了attention是对encoder对重编码，qk完成权重重新计算，v复制重编码

# 为什么self-attention可以替代seq2seq？
- seq2seq最大的问题在于将Encoder端的所有信息压缩到一个固定长度的向量中，并将其作为Decoder端首个隐藏状态的输入，来预测Decoder端第一个单词(token)的隐藏状态。在输入序列比较长的时候，这样做显然会损失Encoder端的很多信息，而且这样一股脑的把该固定向量送入Decoder端，Decoder端不能够关注到其想要关注的信息
- self-attention让源序列和目标序列首先“自关联”起来，这样的话，源序列和目标序列自身的embedding表示所蕴含的信息更加丰富，而且后续的FFN层也增强了模型的表达能力，并且Transformer并行计算的能力是远远超过seq2seq系列的模型

# 维度与点积大小的关系是怎么样的，为什么使用维度的根号来放缩?
- 假设向量 q 和 k 的各个分量是互相独立的随机变量，均值是0，方差是1，那么点积 qk 的均值是0，方差是 dk
- 针对Q和K中的每一维i都有qi和ki相互独立且均值0方差1，不妨记![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9envjoy8oj301h00gdfl.jpg),![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9envuw3p0j301g00ga9t.jpg)
    - E(XY) = E(X)E(Y)=0
    - ![](https://tva1.sinaimg.cn/large/006y8mN6gy1g9enzh4vzvj30gh017t8t.jpg)
    - 所以k维度上的qk方差会为dk，均值为0，用维度的根号来放缩，使得标准化