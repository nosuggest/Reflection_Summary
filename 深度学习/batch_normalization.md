# 你觉得bn过程是什么样的？
- 按batch进行期望和标准差计算
- 对整体数据进行标准化
- 对标准化的数据进行线性变换
    - 变换系数需要学习

# 手写一下bn过程？
- mu = 1.0*np.sum(X,axis = 0)/X.shape\[0]
- Xmu = X - mu
- sq = Xmu**2
- var = 1.0*np.sum(sq,axis=0)/X.shape\[0]
- out = alhpa*(X-Xmu)/np.sqrt(var+eps) + beta

# 知道LN么？讲讲原理
- 和bn过程近似，只是作用的方向是在维度上，而不是batch上
- 这样做的好处就是不会受到batch大小不一致的影响