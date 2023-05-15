参考博客：https://blog.csdn.net/Tink1995/article/details/105080033?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168410960816800180628087%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=168410960816800180628087&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-105080033-null-null.142^v87^control_2,239^v2^insert_chatgpt&utm_term=transformer&spm=1018.2226.3001.4187


https://blog.csdn.net/benzhujie1245com/article/details/117173090?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522168410960816800180628087%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=168410960816800180628087&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-2-117173090-null-null.142^v87^control_2,239^v2^insert_chatgpt&utm_term=transformer&spm=1018.2226.3001.4187  

### Add＆Normalize：

Add，就是在Z的基础上加了一个残差块X，加入残差块X的目的是为了防止在深度神经网络训练中发生退化问题，退化的意思就是深度神经网络通过增加网络的层数，Loss逐渐减小，然后趋于稳定达到饱和，然后再继续增加网络层数，Loss反而增大。
**个人理解：加上一个残差块可以保证深度网络可以出现浅层网络的现象。**

* **为什么要进行Normalize呢？**  
在神经网络进行训练之前，都需要对于输入数据进行Normalize归一化，目的有二：1，能够加快训练的速度。2.提高训练的稳定性。  
* **为什么使用Layer Normalization（LN）而不使用Batch Normalization（BN）呢？**

