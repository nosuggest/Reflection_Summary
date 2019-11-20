# xgboost对比gbdt/boosting Tree有了哪些方向上的优化？
- 显示的把树模型复杂度作为正则项加到优化目标中
- 优化目标计算中用到二阶泰勒展开代替一阶，更加准确
- 实现了分裂点寻找近似算法
    - 暴力枚举
    - 近似算法（分桶）
- 更加高效和快速
    - 数据事先排序并且以block形式存储，有利于并行计算
    - 基于分布式通信框架rabit，可以运行在MPI和yarn上
    - 实现做了面向体系结构的优化，针对cache和内存做了性能优化
    
# xgboost优化目标/损失函数改变成什么样？
- 原始：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mjezeisj307401fmx0.jpg)
    - ![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mnp3fd7j301700idfl.jpg)为泰勒一阶展开，![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mrtqxv0j30480173yc.jpg)
- 改变：![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mldvhz5j30ay01k3yf.jpg)
    - J为叶子结点的个数，![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mo56g1vj300o00e0s1.jpg)为第j个叶子结点中的最优值    
    - ![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mnp3fd7j301700idfl.jpg)为泰勒二阶展开，![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mtjds7qj309r0193yg.jpg)

# xgboost如何使用MAE或MAPE作为目标函数？
MAE:![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mxhhvg8j303l011q2q.jpg)
MAPE:![](https://tva1.sinaimg.cn/large/006y8mN6gy1g94mx7uyuej303f0170sj.jpg)
- 利用可导的函数逼近MAE或MAPE
    - mse
    - Huber loss
    - Pseudo-Huber loss

# xgboost如何寻找分裂节点的候选集？
- 暴力枚举
    - 法尝试所有特征和所有分裂位置，从而求得最优分裂点。当样本太大且特征为连续值时，这种暴力做法的计算量太大
- 近似算法（approx）
    - 近似算法寻找最优分裂点时不会枚举所有的特征值，而是对特征值进行聚合统计，然后形成若干个桶。然后仅仅将桶边界上的特征的值作为分裂点的候选，从而获取计算性能的提升
        - 离散值直接分桶
        - 连续值分位数分桶

# xgboost如何处理缺失值？
- 训练时：缺失值数据会被分到左子树和右子树分别计算损失，选择较优的那一个
- 预测时：如果训练中没有数据缺失，预测时出现了数据缺失，那么默认被分类到右子树

# xgboost在计算速度上有了哪些点上提升？
- 特征预排序
    - 按特征进行存储，每一个block代表一个特征的值，样本在该block中按照它在该特征的值排好序。这些block只需要在程序开始的时候计算一次，后续排序只需要线性扫描这些block即可
    - block可以仅存放样本的索引，而不是样本本身，这样节省了大量的存储空间