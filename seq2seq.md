参考自https://blog.csdn.net/zhuge2017302307/article/details/119979892?ops_request_misc=&request_id=&biz_id=102&utm_term=seq2seq&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-0-119979892.142^v87^control_2,239^v2^insert_chatgpt&spm=1018.2226.3001.4187

## 基础知识储备：

* BOS：begining of sequence，代表序列开始。
* EOS：End of sequence，代表序列结束。
* UNK: 低频词或未在词表中的词
* PAD: 补全字符

* Epoch（时期）：
当一个完整的数据集通过了神经网络一次并且返回了一次，这个过程称为一次>epoch。（也就是说，所有训练样本在神经网络中都 进行了一次正向传播 和一次反向传播 ）
然而，当一个Epoch的样本（也就是所有的训练样本）数量可能太过庞大（对于计算机而言），就需要把它分成多个小块，也就是就是分成多个Batch 来进行训练。

* Batch（批 / 一批样本）：
将整个训练样本分成若干个Batch。

* Batch_Size（批大小）：
每批样本的大小。 样本数量/ 批次数= batch size
batchSize表示批次大小，如bathSize=5，代表模型处理完5个样本后，进行一次前向传播和反向传播;

* Iteration（一次迭代）：
训练一个Batch就是一次Iteration

## 一、RNN
一个RNN包括隐藏状态**h**,一个可选的输出**y**，可变长度输入序列x， X = {x1, x2, … xT}。  
![image](https://img-blog.csdnimg.cn/dcb303c6c2b04d1daaa73b2516ff261b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Yqq5Yqb5LiN6ISx5Y-R6YCJ5omL,size_17,color_FFFFFF,t_70,g_se,x_16)

