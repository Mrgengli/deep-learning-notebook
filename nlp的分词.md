# [LLM中的分词算法与分词器（tokenization & tokenizers）：BPE/WordPiece/ULM & beyond](https://zhuanlan.zhihu.com/p/626080766)

在分词的时候的主要问题：
- 词粒度的词表由于长尾效应可能会非常大，包含很多的稀有词，存储和训练的成本都很高，并且稀有词往往很难学好
- OOV问题，对于词表之外的词无能为力
- 无法处理单词的形态关系和词缀关系：同一个词的不同形态，语义相近，完全当做不同的单词不仅增加了训练成本，而且无法很好的捕捉这些单词之间的关系；同时，也无法学习词缀在不同单词之间的泛化

## 分词的三种粒度：
- word-based tokenizer
- character-based tokenizer
- subword-baed tokenizer


目前有三种主流的Subword分词算法，分别是Byte Pair Encoding (BPE), WordPiece和Unigram Language Model。  

## BPE
[详细讲解地址](https://zhuanlan.zhihu.com/p/448147465)

```python
import re, collections

def get_vocab(filename):
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
