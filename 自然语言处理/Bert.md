# Bert的双向体现在什么地方？
mask+attention，mask的word结合全部其他encoder word的信息

# Bert的是怎样实现mask构造的？
- MLM：将完整句子中的部分字mask，预测该mask词
- NSP：为每个训练前的例子选择句子 A 和 B 时，50% 的情况下 B 是真的在 A 后面的下一个句子， 50% 的情况下是来自语料库的随机句子，进行二分预测是否为真实下一句

# 在数据中随机选择 15% 的标记，其中80%被换位\[mask]，10%不变、10%随机替换其他单词，这样做的原因是什么？
- mask只会出现在构造句子中，当真实场景下是不会出现mask的，全mask不match句型了
- 随机替换也帮助训练修正了\[unused]和\[UNK]
- 强迫文本记忆上下文信息

# 为什么BERT有3个嵌入层，它们都是如何实现的？
- input_id是语义表达，和传统的w2v一样，方法也一样的lookup
- segment_id是辅助BERT区别句子对中的两个句子的向量表示，从\[1,embedding_size]里面lookup
- position_id是为了获取文本天生的有序信息，否则就和传统词袋模型一样了，从\[511,embedding_size]里面lookup

# bert的损失函数？
- MLM:在 encoder 的输出上添加一个分类层,用嵌入矩阵乘以输出向量，将其转换为词汇的维度,用 softmax 计算mask中每个单词的概率
- NSP:用一个简单的分类层将 \[CLS] 标记的输出变换为 2×1 形状的向量,用 softmax 计算 IsNextSequence 的概率
- MLM+NSP即为最后的损失

# 手写一个multi-head attention？
tf.multal(tf.nn.softmax(tf.multiply(tf.multal(q,k,transpose_b=True),1/math.sqrt(float(size_per_head)))),v)

# 长文本预测如何构造Tokens？
- head-only：保存前 510 个 token （留两个位置给 \[CLS] 和 \[SEP] ）
- tail-only：保存最后 510 个token
- head + tail ：选择前128个 token 和最后382个 token（文本在800以内）或者前256个token+后254个token（文本大于800tokens）

# 你用过什么模块？bert流程是怎么样的？
- modeling.py
- 首先定义处理好输入的tokens的对应的id作为input_id,因为不是训练所以input_mask和segment_id都是采取默认的1即可
- 在通过embedding_lookup把input_id向量化，如果存在句子之间的位置差异则需要对segment_id进行处理，否则无操作；再进行position_embedding操作
- 进入Transform模块，后循环调用transformer的前向过程，次数为隐藏层个数，每次前向过程都包含self_attention_layer、add_and_norm、feed_forward和add_and_norm四个步骤
- 输出结果为句向量则取\[cls]对应的向量（需要处理成embedding_size），否则也可以取最后一层的输出作为每个词的向量组合all_encoder_layers\[-1]

# 知道分词模块：FullTokenizer做了哪些事情么？
- BasicTokenizer：根据空格等进行普通的分词
    - 包括了一些预处理的方法：去除无意义词，跳过'\t'这些词，unicode变换，中文字符筛选等等
- WordpieceTokenizer：前者的结果再细粒度的切分为WordPiece
    - 中文不处理，因为有词缀一说：解决OOV

# Bert中如何获得词意和句意？
- get_pooled_out代表了涵盖了整条语句的信息
- get_sentence_out代表了这个获取每个token的output 输出，用的是cls向量
    
# 源码中Attention后实际的流程是如何的？
- Transform模块中：在残差连接之前，对output_layer进行了dense+dropout后再合并input_layer进行的layer_norm得到的attention_output
- 所有attention_output得到并合并后，也是先进行了全连接，而后再进行了dense+dropout再合并的attention_output之后才进行layer_norm得到最终的layer_output

# 为什么要在Attention后使用残差结构？
残差结构能够很好的消除层数加深所带来的信息损失问题

# 平时用官方Bert包么？耗时怎么样？
- 第三方：bert_serving
- 官方：bert_base
- 耗时：64GTesla，64max_seq_length，80-90doc/s
    - 在线预测只能一条一条的入参，实际上在可承受的计算量内batch越大整体的计算性能性价比越高
    
# 你觉得BERT比普通LM的新颖点？
- mask机制
- next_sentence_predict机制

# elmo、GPT、bert三者之间有什么区别？
- 特征提取器：elmo采用LSTM进行提取，GPT和bert则采用Transformer进行提取。很多任务表明Transformer特征提取能力强于LSTM，elmo采用1层静态向量+2层LSTM，多层提取能力有限，而GPT和bert中的Transformer可采用多层，并行计算能力强。
- 单/双向语言模型：GPT采用单向语言模型，elmo和bert采用双向语言模型。但是elmo实际上是两个单向语言模型（方向相反）的拼接，这种融合特征的能力比bert一体化融合特征方式弱。
- GPT和bert都采用Transformer，Transformer是encoder-decoder结构，GPT的单向语言模型采用decoder部分，decoder的部分见到的都是不完整的句子；bert的双向语言模型则采用encoder部分，采用了完整句子