# 变长数据如何处理的？
- input数据中只拿了近20次的点击，部分用户是没有20次的历史行为的，所以我们记录了每一个用户实际点击的次数，在做embedding的时候，我们除以的是真实的history length
    - 20次点击过去一周内的行为，曾经尝试扩大历史点击次数到40，60没有很明显的效果提升
    - 点击行为是处理过的，停留时间过短的click不要
    - 点击行为是处理过的，连续多次的重复点击会去重
    - 点击行为是处理过的，session内的点击次数需要在约定范围内

# input是怎么构造的
- 最近历史20次点击商品id/文章id，如果不足不需要补充
- 最近历史20次点击商品id对应的品牌/文章id对应的类目，如果不足不需要补充
- 最近历史20次点击商品id对应的类别/文章id对应的栏目，如果不足不需要补充
- 最后一次点击商品id/文章id
- 历史上最高频的商品id/文章id
- example age
- user_info:age/gender/地理位置/注册时长
- cross_info:最后一次点击距click时间，最后一次点击商品浏览次数
- phone_info:设备信息，登录状态

# 最后一次点击实际如何处理的？
我们会以日进行切分，每日首次点击的lastclick会以\[unknow]进行替代，隔日的点击不会进行计算

# output的是时候train和predict如何处理的
- train的时候是进行负采样的
- predict的时候是进行的all_embedding dot

# 如何进行负采样的？
- 该次点击时间之前所以的item或者article作为候选集
- 负采样我们会进行剔除，把该次click下的同时show的样本进行剔除后采样
- 均衡采样，不会根据其他样本show time进行加权
    - 为了尽可能多的修正全量样本，尽快达到收敛
    - 为了避免其他推荐产生的交叉影响

# item向量在softmax的时候你们怎么选择的？
是用初始化我们在进行history click embedding过程中使用的初始化的向量，没有在最后层重新构造一个item embedding的结果，实测效果翻倍的要好

# Example Age的理解？
- 官方：upload_time-click_time
    - 希望更倾向于新上视频
- 民间：click_time-now
    - 希望平衡样本构造时间对当前的影响

# 什么叫做不对称的共同浏览（asymmetric co-watch）问题？
item对比nlp问题的时候，上下文信息更适用于文本等固定结果的探索问题（完形填空问题）；而在预测next one这种问题下，只通过上文进行预测，更加合适和合理。浏览信息大概率都是不对称的，而常规的i2i模型的假设都是对称的，上下文都可以用的

# 为什么不采取类似RNN的Sequence model？
在实际的推荐数据获取中，历史点击流收到若干种在线的推荐算法影响，并不全是像自然语言问题一样真实具有序列性

# YouTube如何避免百万量级的softmax问题的？
负采样

# serving过程中，YouTube为什么不直接采用训练时的model进行预测，而是采用了一种最近邻搜索的方法？
工程妥协，避免几百万的item每次都要计算一边，而采取的ANN的邻近搜索加快速度

# Youtube的用户对新视频有偏好，那么在模型构建的过程中如何引入这个feature？
example age

# 在处理测试集的时候，YouTube为什么不采用经典的随机留一法（random holdout），而是一定要把用户最近的一次观看行为作为测试集？
不对称的共同浏览问题，避免引入future information，产生与事实不符的数据穿越

# 整个过程中有什么亮点？有哪些决定性的提升？
- embedding
    - 引入了doc2vec做init
    - 权重共享，没有在softmax处重新构造
- 负采样
    - 限定采样集合在click时间发生之前已经有的item
    - 剔除该次点击click时同时展现的其他item
    - 均衡采样
        - 加快收敛
        - 避免热门商品item的过度影响
- click数据的预处理
    - history click 
        - 停留时间过短的click不要
        - 连续多次的重复点击会去重
        - session内的点击次数需要在约定范围内
    - last click
        - 每日初次点击的lastclick以\[unknown]替代，不做隔日的数据连接
- 在history引入multihead-attention
    - ctr提高了，但是有效点击没有变