# 召回层构造loss和精排层的差异？
- 常规做法：召回侧的损失函数构造除了pointwise外，可以考虑pairwise，其中常见方法为<u,d+>与<u,d->尽可能大，在实际计算的过程中，为了平衡我们认为大于一定程度就提供0loss进行反推/bp，实际实现就是max(0,margin-<u,d+>+<u,d->)，参考svm中的hingle loss构造
- 优化做法：BPR loss=log(1+exp(<u,d->-<u,d+>))。本质上没有限制贡献度的上界，理论上无穷大的差距下也有细微的贡献，更核心的是少一个margin的超参训练

# 离线评估有什么办法？
- 拿Top K召回结果与用户实际点击做交集，然后计算precision/recall，这个实际使用过，可行的，可以考虑top1，top3，top100，top200，top500分别计算得到准召
- 用户实际点击位置与召回结果结合，这个实际也使用过，用户分布不均匀（同时预测100个skn，A用户预测结果匹配到10个，均在前15；B用户预测结果匹配到30个，均在50-100，很难客观描述，A比B更加准确，B比A更加多）影响到位置offset的计算

# 负样本为什么不能用点击未展示？
- 收敛慢，覆盖量少
- 未点击不代表没有兴趣，可能是存在更有偏向的结果
- 只能让模型学习到茧房信息，对全量数据不能很好的学习到
- 通常会选择负采样替代展示未点击
    - 对样本会做上下采样，下采样用(sqrt(z(wi)/0.001)+1)`*`(0.001/z(wi))，上采样用(n(wi)^α)/Σj(n(wj)^α)
    
# 解释一下hard negative？
- 正负样本正常逻辑，中间层样本作为hard negative，增强边界刻画能力，关注模型数据细节。
    - 人为构造，同质未被点击内容作为负样本。
    - 选择召回排序中间部分，前面是正样本，后面是负样本，针对中间的样本可以重复多次训练，强化分类能力
    - 个人使用是10:1，<Embedding-based Retrieval in Facebook Search>中有人提到如下的100:1。hard negative并非要替代easy negative(明确为负样本)，而是easy negative的补充。在数量上，负样本还是以easy negative为主。

# 什么样本是hard和easy的？
通过prob或者loss判断不具备足够的说服力为hard

# 如何处理hard部分？
- 预测结果判断
    - 第一步：构建Positive(pos)和Negative(neg)样本组成的数据集，训练一个分类器(cls)；
    - 第二步：用cls预测neg，得到预测prob>0.5的所有neg对应的feature和prob；
    - 第三步：把第二步得到的feature喂给cls，重新train；
    - 第四步：把第二步和第三步迭代几次；
- 《Training Region-based Object Detectors with Online Hard Example Mining》提出，用loss找出hard example，bp的时候只回传这些hard example的weight更新