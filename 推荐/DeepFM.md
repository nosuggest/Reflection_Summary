# DNN与DeepFM之间的区别?
DNN是DeepFM中的一个部分，DeepFM多一次特征，多一个FM层的二次交叉特征

# Wide&Deep与DeepFM之间的区别?
DeepFM对Wide&Deep中的Wide层进行了优化，增加了交叉特征

# 你在使用deepFM的时候是如何处理欠拟合和过拟合问题的？
- 欠拟合：增加deep部分的层数，增加epoch的轮数，增加learning rate，减少正则化力度
- 过拟合：在deep层直接增加dropout的率，减少epoch轮数，增加更多的数据，增加正则化力度，shuffle数据

# DeepFM怎么优化的？
- embedding向量可以通过FM初始化 
- Deep层可以做优化
    - NFM:把deep层的做NFM类型的处理，其实就是deep层在输入之前也做一个二阶特征的交叉处理和fm层一致
- FM层可以变得交叉更多阶
    - XDeepFM

# 不定长文本数据如何输入deepFM？
- 截断补齐
- 结合文本id+文本长度，在做文本处理之前，先做不等长的sum_pooled的操作

# deepfm的embedding初始化有什么值得注意的地方吗？
- 常规的是Xavier，输出和输出可以保持正态分布且方差相近：np.random.rand(layer\[n-1],layer\[n])*np.sqrt(1/layer\[n-1])
    - relu的情况下通常是HE，保证半数神经元失活的情况下对输出方差影响最小:：np.random.rand(layer\[n-1],layer\[n])*np.sqrt(2/layer\[n-1])
- 文本项目上也可以用预训练好的特征
