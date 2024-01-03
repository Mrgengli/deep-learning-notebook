# NLP Subword三大原理：[BPE、wordpiece、ulm](https://zhuanlan.zhihu.com/p/191648421)

subword与传统分词方法的比较：
* 传统词表示方法无法很好的处理未知或罕见的词汇（OOV 问题）。
* 传统词 tokenization 方法不利于模型学习词缀之间的关系，例如模型学到的“old”, “older”, and “oldest”之间的关系无法泛化到“smart”, “smarter”, and “smartest”。
* Character embedding 作为 OOV 的解决方法粒度太细。
* Subword 粒度在词与字符之间，能够较好的平衡 OOV 问题。
* 目前有三种主流的 Subword 算法，它们分别是：Byte Pair Encoding (BPE)、WordPiece 和 Unigram Language Model。

## 1.[BPE分词原理](https://zhuanlan.zhihu.com/p/448147465)
相关源代码可以参考：[subword-nmt](https://github.com/rsennrich/subword-nmt/tree/master/subword_nmt)

BPE的代码实现：
```python
import re, collections

def get_vocab(filename):
    # defaultdict这个类是一个在字典访问不存在的时候也会返回一个默认的int值
    vocab = collections.defaultdict(int)
    with open(filename, 'r', encoding='utf-8') as fhand:
        for line in fhand:
            words = line.strip().split()
            for word in words:
                vocab[' '.join(list(word)) + ' </w>'] += 1
    return vocab

def get_stats(vocab):
    pairs = collections.defaultdict(int)
    for word, freq in vocab.items():
        symbols = word.split()
        for i in range(len(symbols)-1):
            pairs[symbols[i],symbols[i+1]] += freq
    return pairs

def merge_vocab(pair, v_in):
    v_out = {}
    bigram = re.escape(' '.join(pair))
    p = re.compile(r'(?<!\S)' + bigram + r'(?!\S)')
    for word in v_in:
        w_out = p.sub(''.join(pair), word)
        v_out[w_out] = v_in[word]
    return v_out

def get_tokens(vocab):
    tokens = collections.defaultdict(int)
    for word, freq in vocab.items():
        word_tokens = word.split()
        for token in word_tokens:
            tokens[token] += freq
    return tokens
```

## 2.worldpiece
Google的Bert模型在分词的时候使用的是WordPiece算法。与BPE算法类似，WordPiece算法也是每次从词表中选出两个子词合并成新的子词。与BPE的最大区别在于，如何选择两个子词进行合并：BPE选择频数最高的相邻子词合并，而WordPiece选择能够提升语言模型概率最大的相邻子词加入词表。

![](https://github.com/Mrgengli/deep-learning-notebook/blob/main/worldpiece_image.png)  

从上面的公式，很容易发现，似然值的变化就是两个子词之间的互信息。简而言之，WordPiece每次选择合并的两个子词，他们具有最大的互信息值，也就是两子词在语言模型上具有较强的关联性，它们经常在语料中以相邻方式同时出现。

## 3.[Unigram Language Model (ULM)](https://zhuanlan.zhihu.com/p/191648421)


