| **Title** | Practical Binary Code Similarity Detection with BERT-based Transferable Similarity Learning |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Sunwoo Ahn |
| **Institution** | Seoul National University |
| **Venue** | ACSAC |
| **Year** | 2022 |
| **Paper** | [link](https://doi.org/10.1145/3564625.3567975)| 

# ABSTRACT
1. 基于 `BERT`，并使用带有加权距离向量的二元交叉熵作为损失函数
2. 基于 `Siamese`
3. 单样本学习（少样本学习的特例
4. 具有迁移性和实用性


# CONCLUSION


# FUTURE WORK


# 1. INTRODUCTION 
- 主要贡献点
  1. 基于 `BERT` 和 `Siamese`，使用**加权距离向量和二元交叉熵**
     - 主要贡献点
  2. 开源


# 2. BACKGROUND
## 2.1. Few-shot Learning
1. 核心思想
   - 使模型**学习如何学习**，即重点不是直接识别标签，而是**识别对象之间的相似性或者差异性**

# 4. BINSHOT DESIGN
## 4.1. Binshot Overview
![alt text](<images/2022_B_ACSAC_Practical Binary Code Similarity Detection with BERT-based Transferable Similarity Learning/img.png>)


## 4.2. Preprocessor: Learning Preparation
1. 反汇编：二进制文件 >> 汇编代码
2. 对汇编指令标准化：防止 OOV 问题（常见操作
   - 值得注意的是，两个不同的函数可能在经过标准化后，**形成完全统一的表达**，文中采用的方法是排除这些函数（**值得参考**


## 4.3. Pre-trainer: Model for Assembly
- 与原 `BERT` 相同，使用 `MLM` 进行预训练，但是不进行 `NSP`
1. MLM：与 `BERT` 相同，用于捕捉指令间的上下文关系
2. NSP
   - 函数之间的关系是**由函数调用决定，和其绝对位置没有关系**
 

## 4.4. Fine-tuner: Model for Code Similarity
1. 二元交叉熵损失函数的优点：在推理未见过的函数时，性能更佳 
   - $L=Y\ log\ p(X_i,X_j)+(1-Y)\ log\ (1-p(X_i,X_j))$ 
2. 假设每函数中只存在一个样本，BCSD 任务就变成了**特例的少样本学习**，即 N way 1 次分类问题


# 6. EVALUATION
## 6.3. Transferability
![alt text](<images/2022_B_ACSAC_Practical Binary Code Similarity Detection with BERT-based Transferable Similarity Learning/img-2.png>)



## 6.5. Function Embedding Visualization
![alt text](<images/2022_B_ACSAC_Practical Binary Code Similarity Detection with BERT-based Transferable Similarity Learning/img-1.png>)
1. 经过 `t-SNE` 方法降维后，`BinShot` 生成的相似函数的嵌入向量能够很好的聚合在一起
2. 关于 `hmac_key` 函数，`BinShot` 效果相对较差的原因是其来自**不同的编译器**

# RELATED KNOWLEDGE