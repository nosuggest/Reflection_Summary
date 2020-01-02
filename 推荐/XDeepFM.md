# 选用的原因？
- 类似deepfm和FNN等模型的高阶的特征交互来自于dnn部分，但是这样的特征交互是不可控且隐式的，难以描述的
- 向量级别的特征交互而不是元素级交互
    - 经验上，vector-wise的方式构建的特征交叉关系比bit-wise的方式更容易学习
        - 我也不知道具体好在哪，如果有大佬会可以指导一下，感恩
- 之前用的deepfm在历史数据的拟合上出现了瓶颈：A\["篮球","足球","健身"]，B\["篮球","电脑","蔡徐坤"]，会给A推荐"蔡徐坤"，但是实际上不合理
    - 思路一：改变Memorization为attention网络，强化feature直接的关系，对B进行"电脑"与"蔡徐坤"之间的绑定而不是"篮球"和"蔡徐坤"之间的绑定
    - 思路二：改变Memorization为更优化更合理的低价特征交互，比如DCN或者XDeepFM


# 什么叫显示隐式？什么叫元素级/向量级？什么叫做高阶/低阶特征交互？
- 显示是可以写出feature交互的公式，隐式相反
- 元素级是以feature值交互，向量级是feature向量级点乘处理
- 高阶特征是类似DNN这种多层特征交互，低阶特征交互是FM这种特征单层处理方式

# 简单介绍一下XDeepFm的思想？
- 借鉴了DeepFm的整体结构，保持了两个部分的组合：低阶特征交互+高阶特征交互，**低价特征来记忆高频历史数据场景，高阶特征交互来进行稀疏场景的泛化**
    - 高阶特征交互：DNN
    - 低价特征交互：DCN结构(![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9ihwd6wucj305q00la9v.jpg))
        - 这样的网络结构保证来来自X0的1，2，3...N阶的特征组合
- 借鉴来DCN的交叉网络的特殊结构**自动构造有限高阶交叉特征**：![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9iidr9x2bj306r01lglh.jpg)
- 流程概述：
    - feature embedding
    - 构造\[batch,field,embedding_size]的input，分别进入DNN、CIN、Linear层
    - CIN中：
        - 先记录\[batch,field,embedding_size]作为X0，并切分为embedding`*`\[batch,field,1]份
        - 设置三层隐层，单层结点数为200，单层操作如下（以X1为例）：
            - 获取上一次的layer out：X0，并进行切分：embedding`*`\[batch,field,1]
            - 进行外积：embedding`*`\[batch,field,1]`*`embedding`*`\[batch,field,1]得到\[embedding,batch,field,field]
            - 对\[embedding,batch,field,field]进行压缩\[embedding,batch,field`*`field]，在对结果进行\[1,field`*`field,output_layer]卷积得到缩\[embedding,batch,output_layer]
            - 加偏置项，并进行激活函数处理，完成一轮处理
        - 将若干轮的处理结果按照hidden_size维进行合并，并对embedding维度进行pooling得到\[batch,embedding]的output层即为结果
        - 实际过程中，可以对每层对结果进行采样泛化；可以通过最后层输出的残差连接保证梯度消失等等

# 和DCN比，有哪些核心的变化？
- DCN是bit-wise的，而CIN 是vector-wise的
- DCN每层是1～l+1阶特征，而CIN每层只包含 l+1 阶的组合特征
    - ![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9ihwd6wucj305q00la9v.jpg) 和 ![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9iidr9x2bj306r01lglh.jpg)差异的![](https://tva1.sinaimg.cn/large/006tNbRwgy1g9in06u4m6j300j00f0sh.jpg)导致的

# 时间复杂度多少？
假设CIN和DNN每层神经元/向量个数都为 H ，网络深度为 L
- CIN:O(m`*`L`*`H`*`H)
- DNN:O(m`*`D`*`H+L`*`H`*`H)