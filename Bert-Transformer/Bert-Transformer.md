BERT-Transformer
----
01. 概述  
Transformer(simple network architecture based on attention mechanisms)
BERT(Bidirectional Encoder Representations from Transformers)
02. embedding  
Transformer: token embedding + position embedding
BERT: token embedding + position embedding + segment embedding
```python
# tf实现
# 位置embedding，基于序列id随机初始化
position_embeddings = tf.get_variable(
        name=position_embedding_name,
        shape=[max_position_embeddings, width],
        initializer=create_initializer(initializer_range))
```
03. transformer encoder  
(1) one signel encoder = multi-head-attention + 
(2) self-attention 中的变量变换  
self-attention 中的query&key&value 由一份输入经过三次线性变换的到
```python
Q = tf.layers.dense(inputx, units=d_model)  # [batch,sequence_length,d_model]
K = tf.layers.dense(inputx, units=d_model)  # [batch,sequence_length,d_model]
V = tf.layers.dense(inputx, units=d_model)  # [batch,sequence_length,d_model]
```
self-attention  
```python
# attention_weight = softmax(QV^T/sqrt(d_model))
# attention-output = attention_weight.V
a_weight=tf.matmul(Q,K,transpose_b=True)  # [batch,h,sequence_length,sequence_length]
a_weight=a_weight*(1.0/tf.sqrt(tf.cast(d_model,tf.float32)))  # [batch,h,sequence_length，sequence_length]
```
(3) multi-head-attention 中的多头划分
```python
# hidden_size % num_attention_heads == 0
# split Q,K,V h*d_k = d_model
# 4x128->split->[4x64, 4x64]->stack->2x4x64
Q_heads = tf.stack(tf.split(Q, h, axis=2), axis=1)  # [batch,h,sequence_length,d_k]
K_heads = tf.stack(tf.split(K, h, axis=2), axis=1)  # [batch,h,sequence_length,d_k]
V_heads = tf.stack(tf.split(V, h, axis=2), axis=1)  # [batch,h,sequence_length,d_k]
```
(4) position_wise_feed_forward 前馈网络  
position_wise_feed_forward 为2层全连接，实现上等价于1x1卷积
第一层激活函数为relu 第二层没有激活函数
```python
def position_wise_feed_forward_fn(self):
    """
    x:       [batch,sequence_length,d_model]
    :return: [batch,sequence_length,d_model]
    """
    output=None
    #1.conv1
    input=tf.expand_dims(self.x,axis=3) #[batch,sequence_length,d_model,1]
    # conv2d.input:       [None,sentence_length,embed_size,1]. filter=[filter_size,self.embed_size,1,self.num_filters]
    # output with padding:[None,sentence_length,1,1]
    output_conv1=tf.layers.conv2d(
        input,filters=self.d_ff,kernel_size=[1,self.d_model],padding="VALID",
        name='conv1',kernel_initializer=self.initializer,activation=tf.nn.relu
    )
    output_conv1 = tf.transpose(output_conv1, [0,1,3,2])
    print("output_conv1:",output_conv1)
    #2.conv2
    output_conv2 = tf.layers.conv2d(
        output_conv1,filters=self.d_model,kernel_size=[1,self.d_ff],padding="VALID",
        name='conv2',kernel_initializer=self.initializer,activation=None
    )
    output=tf.squeeze(output_conv2) #[batch,sequence_length,d_model]
    return output #[batch,sequence_length,d_model]
```
(5) encoder 输入输出总览
```python
inputx_emb = embedding_layer(encoder_input)
inputx_att = self_attention_layer(inputx_emb)
inputx_att = inputx_att + inputx_emb
position_out = position_wise_input_layer(inputx_att)
encoder_output = position_out + inputx_att
```

FAQ
---
01. embedding层相关  
(1) transformer中的位置编码？  
使用正余弦函数表示的位置矩阵，可以表示出相对距离  
(2) BERT中的位置编码？  
基于序列位置的随机表示，一个位置对应一个embdeeding(BERT数据更加充足)  
(3) BERT表示超长文本？  
(a) 源代码中直接截取最大长度512 (b) 层次化&主干提取  

02. self-attention层相关  
(1) self-attention机制的意义？  
用文本中的其它词来增强目标词的语义表示，从而更好的利用上下文的信息  
(2) 为何使用多头？  
https://www.zhihu.com/question/341222779  
多头保证了transformer可以注意到不同子空间的信息，捕捉到更加丰富的特征信息。其实本质上是论文原作者发现这样效果确实好  
(3) 为何要缩放sqrt(dm)?  
比较大的输入会使得softmax的梯度变得很小  
(4) Q和K使用不同的权重矩阵生成?  
打破对称性，增加泛化能力  
(5) self-attention 的时间复杂度？  
O(n^2d)  
(6) 在计算attention score的时候如何对padding做mask操作？  
对需要mask的位置设为负无穷，再对attention score进行相加  

03. 前馈层&归一化层相关  
(1) 前馈层的结构和意义？  
强化位置信息的表示和特征的提取(更好的提取特征)  
(2) 归一化层的结构和意义？  
归一化是LayerNorm，在hiddensize维度作变换将数据分布拉会正太分布，方式梯度消失和爆炸  

04. 为什么BERT在第一句前会加一个[CLS]标志?  
[CLS]符号会更“公平”地融合文本中各个词的语义信息，从而更好的表示整句话的语义  
