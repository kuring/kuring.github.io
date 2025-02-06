# 基本介绍

Word2Vec 诞生于 2013 年，为一种将词汇表中的每个词都表示成固定向量长度的算法。

Word2Vec 有两种实现方法，一种是给定目标词来预测上下文，叫做 Skip-Gram 模型；另外一种是给定上下文来预测目标词。

# Skip-Gram 模型的代码实现

Skip-Gram 模型的任务为给定目标词来预测上下文。

## 构建语料库

```python
# 定义一个句子列表，后面会用这些句子来训练 CBOW 和 Skip-Gram 模型  
sentences = ["Kage is Teacher", "Mazong is Boss", "Niuzong is Boss",  
             "Xiaobing is Student", "Xiaoxue is Student",]  
# 将所有句子连接在一起，然后用空格分隔成多个单词  
words = ' '.join(sentences).split()  
# 构建词汇表，去除重复的词  
word_list = list(set(words))  
# 创建一个字典，将每个词映射到一个唯一的索引  
word_to_idx = {word: idx for idx, word in enumerate(word_list)}  
# 创建一个字典，将每个索引映射到对应的词  
idx_to_word = {idx: word for idx, word in enumerate(word_list)}  
voc_size = len(word_list) # 计算词汇表的大小  
print(" 词汇表：", word_list) # 输出词汇表  
print(" 词汇到索引的字典：", word_to_idx) # 输出词汇到索引的字典  
print(" 索引到词汇的字典：", idx_to_word) # 输出索引到词汇的字典  
print(" 词汇表大小：", voc_size) # 输出词汇表大小
```

输出如下结果
```
词汇表： ['Kage', 'is', 'Boss', 'Xiaobing', 'Mazong', 'Student', 'Xiaoxue', 'Niuzong', 'Teacher']
 词汇到索引的字典： {'Kage': 0, 'is': 1, 'Boss': 2, 'Xiaobing': 3, 'Mazong': 4, 'Student': 5, 'Xiaoxue': 6, 'Niuzong': 7, 'Teacher': 8}
 索引到词汇的字典： {0: 'Kage', 1: 'is', 2: 'Boss', 3: 'Xiaobing', 4: 'Mazong', 5: 'Student', 6: 'Xiaoxue', 7: 'Niuzong', 8: 'Teacher'}
 词汇表大小： 9
```
## 生成 Skip-Gram 数据
```python
# 生成 Skip-Gram 训练数据  
def create_skipgram_dataset(sentences, window_size=2):  
    data = [] # 初始化数据  
    for sentence in sentences: # 遍历句子  
        sentence = sentence.split()  # 将句子分割成单词列表  
        for idx, word in enumerate(sentence):  # 遍历单词及其索引  
            # 获取相邻的单词，将当前单词前后各 N 个单词作为相邻单词  
            for neighbor in sentence[max(idx - window_size, 0):   
                        min(idx + window_size + 1, len(sentence))]:  
                if neighbor != word:  # 排除当前单词本身  
                    # 将相邻单词与当前单词作为一组训练数据  
                    data.append((neighbor, word))  
    return data  
# 使用函数创建 Skip-Gram 训练数据
skipgram_data = create_skipgram_dataset(sentences)  
# 打印未编码的 Skip-Gram 数据样例 
print("Skip-Gram 数据样例（未编码）：", skipgram_data)
```
得到如下结果: 
```
Skip-Gram 数据样例（未编码）： [('is', 'Kage'), ('Teacher', 'Kage'), ('Kage', 'is'), ('Teacher', 'is'), ('Kage', 'Teacher'), ('is', 'Teacher'), ('is', 'Mazong'), ('Boss', 'Mazong'), ('Mazong', 'is'), ('Boss', 'is'), ('Mazong', 'Boss'), ('is', 'Boss'), ('is', 'Niuzong'), ('Boss', 'Niuzong'), ('Niuzong', 'is'), ('Boss', 'is'), ('Niuzong', 'Boss'), ('is', 'Boss'), ('is', 'Xiaobing'), ('Student', 'Xiaobing'), ('Xiaobing', 'is'), ('Student', 'is'), ('Xiaobing', 'Student'), ('is', 'Student'), ('is', 'Xiaoxue'), ('Student', 'Xiaoxue'), ('Xiaoxue', 'is'), ('Student', 'is'), ('Xiaoxue', 'Student'), ('is', 'Student')]
```

数组中的第一个元素为目标词，第二个目标词的上下文词。

## One-Hot 编码

```python
# 定义 One-Hot 编码函数  
import torch # 导入 torch 库  
def one_hot_encoding(word, word_to_idx):      
    tensor = torch.zeros(len(word_to_idx)) # 创建一个长度与词汇表相同的全 0 张量    
tensor[word_to_idx[word]] = 1  # 将对应词的索引设为 1    return tensor  # 返回生成的 One-Hot 向量  
# 展示 One-Hot 编码前后的数据  
word_example = "Teacher"  
print("One-Hot 编码前的单词：", word_example)  
print("One-Hot 编码后的向量：", one_hot_encoding(word_example, word_to_idx))  
# 展示编码后的 Skip-Gram 训练数据样例  
print("Skip-Gram 数据样例（已编码）：", [(one_hot_encoding(context, word_to_idx),   
          word_to_idx[target]) for context, target in skipgram_data[:3]])
```

输出结果：
```
One-Hot 编码前的单词： Teacher
One-Hot 编码后的向量： tensor([0., 0., 1., 0., 0., 0., 0., 0., 0.])
Skip-Gram 数据样例（已编码）： [(tensor([0., 0., 0., 0., 0., 0., 0., 0., 1.]), 4), (tensor([0., 0., 1., 0., 0., 0., 0., 0., 0.]), 4), (tensor([0., 0., 0., 0., 1., 0., 0., 0., 0.]), 8)]
```

# CBOW 模型的代码实现