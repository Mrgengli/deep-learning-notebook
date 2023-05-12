### 注意力机制
* 软性注意力机制：在选择信息的时候，计算N个输入的加权平均，再输入到神经网络中计算
* 硬性注意力机制：选择输入序列某一个位置上的信息，比如随机选择一个信息或者选择概率最高的信息

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
