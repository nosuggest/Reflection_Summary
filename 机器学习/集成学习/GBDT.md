# 介绍一下Boosting的思想？
- 初始化训练一个弱学习器，初始化下的各条样本的权重一致
- 根据上一个弱学习器的结果，调整权重，使得错分的样本的权重变得更高
- 基于调整后的样本及样本权重训练下一个弱学习器
- 预测时直接串联综合各学习器的加权结果

# 最小二乘回归树的切分过程是怎么样的？
- 回归树在每个切分后的结点上都会有一个预测值，这个预测值就是结点上所有值的均值
- 分枝时遍历所有的属性进行二叉划分，挑选使平方误差最小的划分属性作为本节点的划分属性
- 属性上有多个值，则需要遍历所有可能的属性值，挑选使平方误差最小的划分属性值作为本属性的划分值
- 递归重复以上步骤，直到满足叶子结点上值的要求

# 有哪些直接利用了Boosting思想的树模型？
adaboost，gbdt等等

# gbdt和boostingtree的boosting分别体现在哪里？
- boostingtree利用基模型学习器，拟合的是mse（回归）或者指数损失函数（分类）
- gbdt利用基模型学习器，拟合的是当前模型与标签值的损失函数的负梯度

# gbdt的中的tree是什么tree？有什么特征？
Cart tree，但是都是回归树

# 常用回归问题的损失函数？
- mse:![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94bmopzfjj303s011t8i.jpg)
    - 负梯度：y-h(x)
    - 初始模型F0由目标变量的平均值给出
- 绝对损失:![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94bne7cipj303400q742.jpg)
    - 负梯度：sign(y-h(x))
    - 初始模型F0由目标变量的中值给出
- Huber损失：mse和绝对损失的结合
    - 负梯度：y-h(x)和sign(y-h(x))分段函数
    - 它是MSE和绝对损失的组合形式，对于远离中心的异常点，采用绝对损失，其他的点采用MSE，这个界限一般用分位数点度量

# 常用分类问题的损失函数？
- 对数似然损失函数
    - 二元且标签y属于{-1,+1}：𝐿(𝑦,𝑓(𝑥))=𝑙𝑜𝑔(1+𝑒𝑥𝑝(−𝑦𝑓(𝑥)))
        - 负梯度：y/(1+𝑒𝑥𝑝(−𝑦𝑓(𝑥)))
    - 多元：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94cj1yce6j306x01iglg.jpg)
- 指数损失函数:𝐿(𝑦,𝑓(𝑥))=𝑒𝑥𝑝(−𝑦𝑓(𝑥))
    - 负梯度：y·𝑒𝑥𝑝(−𝑦𝑓(𝑥))
除了负梯度计算和叶子节点的最佳负梯度拟合的线性搜索，多元GBDT分类和二元GBDT分类以及GBDT回归算法过程相同

# 什么是gbdt中的损失函数的负梯度？
![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94afa5h1lj3060017mx0.jpg)

当loss函数为均方误差![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94ajgymkuj303w011jr6.jpg)，gbdt中的残差的负梯度的结果y-H(x)正好与boostingtree的拟合残差一致

# 如何用损失函数的负梯度实现gbdt？
- 利用![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94afa5h1lj3060017mx0.jpg)可以计算得到x对应的损失函数的负梯度![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94ars964uj301h00ijr5.jpg),据此我们可以构造出第t棵回归树，其对应的叶子结点区域![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94atb0k7zj300p00i3y9.jpg)j为叶子结点位置
- 构建回归树的过程中，需要考虑找到特征A中最合适的切分点，使得切分后的数据集D1和D2的均方误差最小![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94b0klia2j30ee017aa0.jpg)
- 针对每一个叶子节点里的样本，我们求出使损失函数最小，也就是拟合叶子节点最好的输出值𝑐𝑡𝑗,![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94b5ffnyoj308g0173yd.jpg)
    - 首先，根据feature切分后的损失均方差大小，选取最优的特征切分
    - 其次，根据选定的feature切分后的叶子结点数据集，选取最使损失函数最小，也就是拟合叶子节点最好的输出值
    - 这样就完整的构造出一棵树：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94bfr5cn5j303d01kjr6.jpg)
- 本轮最终得到的强学习器的表达式如下：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94binx5prj307o01k0sl.jpg)

# 拟合损失函数的负梯度为什么是可行的？
- 泰勒展开的一阶形式：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94h6rkexqj305x00ijr7.jpg)
- m轮树模型可以写成：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94h8zovulj305v00iglf.jpg)
    - 对![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94hak9e26j303n00idfm.jpg)进行泰勒展开：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94hd5sz8vj309400kglg.jpg),其中m-1轮对残差梯度为![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94hey2xksj3052017a9w.jpg)
- 我们拟合了残差的负梯度，![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94hi3epnaj302r00kmwx.jpg),所以![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94hd5sz8vj309400kglg.jpg)内会让损失向下降对方向前进

# 即便拟合损失函数负梯度是可行的，为什么不直接拟合残差？ 拟合负梯度好在哪里？
- 前者不用残差的负梯度而是使用残差，是全局最优值，后者使用的是 局部最优方向（负梯度）*步长（𝛽）
- 依赖残差进行优化，损失函数一般固定为反映残差的均方差损失函数，因此 当均方差损失函数失效（该损失函数对异常值敏感）的时候，换了其他一般的损失函数，便很难得到优化的结果。同时，因为损失函数的问题，Boosting Tree也很难处理回归之外问题。 而后者使用梯度下降的方法，对于任意可以求导的损失函数它都可以处理

# Shrinkage收缩的作用？
每次走一小步逐渐逼近结果的效果，要比每次迈一大步很快逼近结果的方式更容易得到精确值，即它不完全信任每一棵残差树，认为每棵树只学到了真理的一部分累加的时候只累加了一小部分多学几棵树来弥补不足。 这个技巧类似于梯度下降里的学习率
- 原始：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94i9bokjzj306g00iglf.jpg)
- Shrinkage：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94iawlfq3j307600it8j.jpg)

# feature属性会被重复多次使用么？
会，同时因为特征会进行多次使用，特征用的越多，则该特征的重要性越大

# gbdt如何进行正则化的？
- 子采样
    - 每一棵树基于原始原本的一个子集进行训练
    - rf是有放回采样，gbdt是无放回采样
    - 特征子采样可以来控制模型整体的方差
- 利用Shrinkage收缩，控制每一棵子树的贡献度
- 每棵Cart树的枝剪

# 为什么集成算法大多使用树类模型作为基学习器？或者说，为什么集成学习可以在树类模型上取得成功？
- 对数据的要求比较低，不需要强假设，不需要数据预处理，连续离散都可以，缺失值也能接受
- bagging，关注于提升分类器的泛化能力
- boosting，关注于提升分类器的精度

# gbdt的优缺点？
优点：
- 数据要求比较低，不需要前提假设，能处理缺失值，连续值，离散值
- 使用一些健壮的损失函数，对异常值的鲁棒性非常强
- 调参相对较简单

缺点：
- 并行化能力差

# gbdt和randomforest区别？
- 相同：
    - 都是多棵树的组合
- 不同：
    - RF每次迭代的样本是从全部训练集中有放回抽样形成的，而GBDT每次使用全部样本
    - gbdt对异常值比rf更加敏感
    - gbdt是串行，rf是并行
    - gbdt是cart回归树，rf是cart分类回归树都可以
    - gbdt是提高降低偏差提高性能，rf是通过降低方差提高性能
    - gbdt对输出值是进行加权求和，rf对输出值是进行投票或者平均
    
# GBDT和LR的差异？
- 从决策边界来说，线性回归的决策边界是一条直线，逻辑回归的决策边界是一条曲线，而GBDT的决策边界可能是很多条线
- 当在高维稀疏特征的场景下，LR的效果一般会比GBDT好。因为LR有参数惩罚，GBDT容易造成过拟合