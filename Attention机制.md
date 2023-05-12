### 注意力机制
* 软性注意力机制：在选择信息的时候，计算N个输入的加权平均，再输入到神经网络中计算
* 硬性注意力机制：选择输入序列某一个位置上的信息，比如随机选择一个信息或者选择概率最高的信息

## 一、软性注意力机制的数学原理

### (一)普通模式

#### 注意力值计算
* 1.计算输入信息的注意力分布

X：输入信息向量（信息存储器） 

Q：查询向量，用来查询并选择X中的某些信息  

Z：是在[1:N]中的值表示被选择信息的索引位置，也就是说如果Z=i表示选择了第i个输入信息  

选择第i个输入信息的概率![image](https://private.codecogs.com/gif.latex?%5Calpha%20_i)为：  

![image](https://img-blog.csdnimg.cn/img_convert/4bf30cbc30ecbda216c0537bd250da51.png)

![image](https://private.codecogs.com/gif.latex?%5Calpha%20_i)构成的概率向量就称为注意力分布，  
![image](https://private.codecogs.com/gif.latex?s%28x_i%20%2C%20q%29)是注意力打分函数（其实就是用来计算X和Q的相关性），有以下几种形式：  
![image](https://img-blog.csdnimg.cn/img_convert/2cbbdddadd915e286f923c19b4c589c7.png)

* 2.用注意力分布对X进行加权平均  

注意力分布![image](https://private.codecogs.com/gif.latex?%5Calpha%20_i)表示在给定查询Q时，输入信息向量X中第i个信息与查询q的相关程度。采用“软性”信息选择机制给出查询所得的结果，就是用加权平均的方式对输入信息进行汇总，得到Attention值![image](https://img-blog.csdnimg.cn/img_convert/b4c61ed182710f90242570097f8712ef.png)：  

下图是计算Attention值的过程图：  
![image](https://img-blog.csdnimg.cn/img_convert/25d76821203a091d77ebe83c86560caf.png)

### （二）键值对注意力模式

我们用键值对来表示输入信息，那么N个输入信息就表示为：![image](https://private.codecogs.com/gif.latex?%28K%2C%20V%29%3D%20%5B%28k_1%2Cv_1%29%2C%28k_2%2Cv_2%29%2C...%2C%28k_N%2Cv_N%29%5D)  

其中“键”用来计算注意分布![image](https://private.codecogs.com/gif.latex?%5Calpha%20_i)，“值”用来计算聚合信息。

那么就可以将注意力机制看做是一种软寻址操作：把输入信息X看做是存储器中存储的内容，元素由地址Key（键）和值Value组成，当前有个Key=Query的查询，目标是取出存储器中对应的Value值，即Attention值。而在软寻址中，并非需要硬性满足Key=Query的条件来取出存储信息，而是通过计算Query与存储器内元素的地址Key的相似度来决定，从对应的元素Value中取出多少内容。每个地址Key对应的Value值都会被抽取内容出来，然后求和，这就相当于由Query与Key的相似性来计算每个Value值的权重，然后对Value值进行加权求和。加权求和得到最终的Value值，也就是Attention值。

具体的计算可以分为三个步骤：  

* 1.根据Query和Key计算二者的相似度。可以用上面所列出的加性模型、点积模型或余弦相似度来计算，得到注意力得分si  
![image](https://img-blog.csdnimg.cn/img_convert/e2e1f5f6e65dd155a98dcff71ad3aa61.png)

* 2.用softmax函数对注意力得分进行数值转换。一方面可以进行归一化，得到所有权重系数之和为1的概率分布，另一方面可以用softmax函数的特性突出重要元素的权重；  
![image](https://img-blog.csdnimg.cn/img_convert/f5eee656fcb20596f22c9ba6d6be26be.png)   
* 3.根据权重系数对Value进行加权求和：  
![image](https://img-blog.csdnimg.cn/img_convert/61be7f528d360edcb25bda67c468c6ea.png)

![image](https://img-blog.csdnimg.cn/img_convert/bc1f5944c43dd88d4017470937ec8bdc.png)  

则上面的公式就可以整理为：
![image](https://img-blog.csdnimg.cn/img_convert/922c81004fbe0e8f4f76aa9e0a2ddeeb.png)

#### 上面就是软性注意力机制的数学原理



## 二、软性注意力机制与Encoder-Decoder框架   
注意力机制是一种通用的思想，不依赖于特定的框架，目前主要应用在Encoder-Decoder框架结合起来使用

以下是没有引入注意力机制的RNN Encoder-Decoder框架：
![image](https://img-blog.csdnimg.cn/img_convert/fce275576e9f01468aebedf2b5a04f3e.png)  

下面我们来对比没有加入注意力机制的模型和加入注意力机制的模型

### （一）未加入注意力机制的RNN Encoder-Decoder  

未加入注意力机制的RNN Encoder-Decoder框架在处理序列数据时，可以做到先用编码器把**长度不固定**的序列X编码成长度固定的**向量表示C**，再用解码器把这个向量表示解码为另一个**长度不固定的序列y**，输入序列X和输出序列y的长度可能是不同的。

《Learning phrase representations using RNN encoder-decoder for statistical machine translation》这篇论文提出了一种RNN Encoder-Decoder的结构，如下图。除外之外，这篇文章的牛逼之处在于首次提出了GRU(Gated Recurrent Unit)这个常用的LSTM变体结构。  
![image](https://img-blog.csdnimg.cn/img_convert/d3aeb9c4f21c0d787b0eef79507198ec.png)  






