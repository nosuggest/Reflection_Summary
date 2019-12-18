# 变长数据如何处理的？
- input数据中只拿了近20次的点击，部分用户是没有20次的历史行为的，所以我们记录了每一个用户实际点击的次数，在做embedding的时候，我们除以的是真实的history length
    - 20次点击过去一周内的行为，曾经尝试扩大历史点击次数到40，60没有很明显的效果提升
    - 点击行为是处理过的，一个session内只有一次点击的行为不会被记录
    - 点击行为是处理过的，连续多次的重复点击会进行处理

# input是怎么构造的
- 最近历史20次点击商品id/文章id，如果不足不需要补充
- 实际历史点击次数
- 最后一次点击商品id/文章id
- example age:商品/文章上线时长
- user_info:age/gender/地理位置/注册时长
- cross_info:最后一次点击距click时间，最后一次点击商品浏览次数
- phone_info:设备信息，登录状态

# 最后一次点击实际如何处理的？
我们会以日进行切分，每日首次点击的lastclick会以\[unknow]进行替代，隔日的点击不会进行计算

# output的是时候train和predict如何处理的
- train的时候是进行负采样的
- predict的时候是进行的all_embedding dot

# 如何进行负采样的？
- 该次点击时间之前所以的item或者article作为代选集
- 负采样我们会进行剔除，把该次click下的同时show的样本进行剔除后采样
- 均衡采样，不会根据其他样本show time进行加权
    - 为了尽可能多的修正全量样本，尽快达到收敛
    - 为了避免其他推荐产生的交叉影响

# item向量在softmax的时候你们怎么选择的？
是用初始化我们在进行history click embedding过程中使用的初始化的向量，没有在最后层重新构造一个item embedding的结果，实测效果翻倍的要好

# Example Age的理解？
实际使用的时候采取的是item上线时间-now的差值，article发表的时间-now的差值，并进行归一化。作用主要是为了平衡因为文章/商品的存量时间对当前选择的一个影响

# 什么叫做不对称的共同浏览（asymmetric co-watch）问题？
item对比nlp问题的时候，上下文信息更适用于文本等固定结果的探索问题（完形填空问题）；而在预测next one这种问题下，只通过上文进行预测，更加合适和合理。浏览信息大概率都是不对称的，而常规的i2i模型的假设都是对称的，上下文都可以用的

# 整个过程中有什么亮点？有哪些决定性的提升？
- 共享item embedding vector
- 加入user feature和cross feature且对feature采取了归一化，归一化方式中均方法+log法对长尾数据有效的标准化了
- 负样本的选取
- attention所起到的作用并不是很明显，尝试了传统的DIN中的attention和传统的Transform中的attention，效果一般
    - 这点其实违反直觉，可能原因是模型对负反馈没有很好的建模