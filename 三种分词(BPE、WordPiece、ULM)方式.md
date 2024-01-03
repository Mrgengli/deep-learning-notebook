# NLP Subword三大原理：BPE、wordpiece、ulm

subword与传统分词方法的比较：
* 传统词表示方法无法很好的处理未知或罕见的词汇（OOV 问题）。
* 传统词 tokenization 方法不利于模型学习词缀之间的关系，例如模型学到的“old”, “older”, and “oldest”之间的关系无法泛化到“smart”, “smarter”, and “smartest”。
* Character embedding 作为 OOV 的解决方法粒度太细。
* Subword 粒度在词与字符之间，能够较好的平衡 OOV 问题。
* 目前有三种主流的 Subword 算法，它们分别是：Byte Pair Encoding (BPE)、WordPiece 和 Unigram Language Model。

## 1.[BPE分词原理](https://zhuanlan.zhihu.com/p/448147465)

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


