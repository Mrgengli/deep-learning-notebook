可参考：https://www.zhihu.com/question/347678607

1.可以通过数据训练学习得到positional encoding，类似于训练学习词向量，goole在之后的bert中的positional encoding便是由训练得到地。  

2.《Attention Is All You Need》论文中Transformer使用的是正余弦位置编码。位置编码通过使用不同频率的正弦、余弦函数生成，然后和对应的位置的词向量相加，位置向量维度必须和词向量的维度一致。过程如上图，PE（positional encoding）计算公式如下：
