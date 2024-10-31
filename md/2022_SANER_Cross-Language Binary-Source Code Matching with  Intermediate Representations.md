| **Title** | Cross-Language Binary-Source Code Matching with Intermediate Representations  |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Yi Gui |
| **Institution** | Huazhong University of Science and Technology  |
| **Venue** | SANER |
| **Year** | 2022 |

# ABSTRACT
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations.png>)
- 相关工作
  - 目前只能实现二进制代码和单一源代码进行匹配
- 本文方法
  - 提出了**二进制代码跨语言进行源代码**匹配的问题，并提出了 XLIR 解决了这个问题
    - 基于 **TRANSFORMER**
    - 学习**二进制和源代码的 IR**
- 实验
  - 跨语言的二进制 - 源代码匹配
  - 跨语言的源代码匹配
  - 以上两者都 XLIR 都优于现有方法


# 1. INTRODUCTION
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-1.png>)
- 现有方法
  - 最近提出的二进制和源代码匹配大多是针对单一语言
  - 方法是使用编码器学习二进制代码和源代码的语义特征，并将它们转换成向量，再使用相似性约束（常用三元损失函数），最后实现它们的匹配
- 现有挑战
  - 现有的方法通常只能针对某一种特定语言进行二进制代码和源代码的匹配
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-2.png>)
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-3.png>)
- 目前方式是使用端到端的方式对齐二进制代码和源代码
  - 由神经网络自动实现代码间的语义对齐，即直接在同一个向量空间中表达出来
- IR 具有**独立于编程语言和目标架构**的特性，这样可以有效的降低源代码和二进制代码间的语义差距
- IR 中语义相似性的体现：while 循环在转换成 LLVM-IR后，它**具有相似的结构**，循环的控制结构、变量操作和跳转逻辑
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-4.png>)
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-5.png>)
- 本文贡献点
  - 提出`XLIR`
    - 基于 TRANSFORMER
    - 使用 LLVM-IR
    - 解决了二进制代码和多种源代码的匹配问题
  - 提出了一个新的跨语言源代码克隆识别数据集
- `XLIR` 的训练过程
  - 预训练 >> 嵌入向量 >> Triple Loss 学习
- 结果
  - `XLIR` 的性能明显优于其他 SOTA 模型
  - 表明 LLVM-IR 可以有效的减少二进制代码和源代码的语义差距


# 2. MOTIVATION
## 2.1. Cross-Language Code Clone Detection
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-6.png>)
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-7.png>)
- 跨语言代码识别的核心是虽然是由不同的编程语言实现的，但是它们都具有相似的语义

## 2.2. Cross-Language Binary-Source Code Matching
- 解释二进制代码跨语言匹配源代码的必要性


# 3. PRELIMINARIES
## 3.1. INTERMEDIATE REPRESENTATION
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-8.png>)
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-9.png>)
- 编译过程
  - source code >> IR >> Binary Code
- 特点
  - **独立**于编程语言和目标构架
  - **保留**了程序的**语义**
- 本文使用的是 `LLVM-IR`，and `end to end`
- 步骤
  - Source Code && Binary Code >> IR >> Embedding Vector (Using Encoder) >> Similarity


## 3.2. Code Embedding
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-10.png>)
- 代码嵌入
  - 目的
    - 将程序语义（特征）转换成向量表示


## 3.3. Problem Formulation
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-11.png>)
- 将二进制代码和源代码映射到同一个向量空间中后，成对的源代码和二进制代码应该尽可能的接近，否则，则应该尽可能的远离
- 公式 1 表示了不使用任何处理，直接将两者映射进同一个向量空间的方式，但两者的**差距过大**，效果并不好
- 本文先将 $S$ 和 $B$ 转换成 **IR**，即 $S_r$ 和 $B_r$；这样它们就**更加接近**，再将它们映射到共同的向量空间中，如公式 2


# 4. CROSS-LANGUAGE BINART-SOURCE CODE MATCHING
## 4.1. An Overview
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-12.png>)
- 模型的训练步骤
  - 1. Source & Binary >> IR
  - 2. IR Embedding
    - 将 IR 输入到预训练的 BERT 中，转换成向量
  - 3. Model Learning
    - 映射到同一个向量空间，学习相似性
- 模型推理
  - 如果输出的相似度大于阈值，则被认为两者是匹配的


## 4.2. Transforming Source and Binary Code into IRs
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-13.png>)
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-14.png>)
- 文章选择的源代码范围为 C、C++ 和 JAVA
- 选择 LLVM-IR 的原因
  - 源无关：不同的编程语言保持相似的 IR 结构
  - 目标无关
  - LLVM- IR 的认可度高
- 转换成 LLVM-IR 的工具
  - C,C++: LLVM-Clang
  - Java: Jlang, Polyglot
  - Binary: RetDec
- LLVM-IR 的形式选择
  - bitcode format: 效率更高


## 4.3. Transformer-based LLVM-IR Embedding
- 这部分是继承了 Transformer 和 BERT 的工作
  - Transformer 模型的解释
  - IR-BERT 的预训练，在大规模的 LLVM-IR 数据集上训练 MLM


## 4.4. Model Learning
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-15.png>)
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-16.png>)
- 三元组实例
  - $<b, s+, s->$
  - $b$ 代表 二进制代码
  - $s+$ 表示与 $b$ 语义相近的源代码，正样本
  - $s-$ 表示随机选择的，与 $b$ 无关的源代码，负样本
- 相似度计算
  - 两者语义相似，余弦相似度值更高；否则，更低
- 损失函数
  - 确保正样本的相似度比负样本更高，且两者的差距至少为$\alpha$，文章选取 $\alpha=0.06$
  - $Loss=\sum max(\alpha-sim(b,s^+)+sim(b,s^-)$


## 4.5. Code Matching
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-17.png>)
- 模型推理
  - 将源代码和二进制代码输入到模型，得到它们的向量表示
  - 计算余弦相似度
  - 相似度大于 80% 则认为两者是匹配的

# 5. EXPERIMENTAL SETUP
## 5.2. Evaluated Tasks and Dataset
1. Cross-Language Source-Source Code Matching
   - 数据集为 CLCDSA
   - 筛选出 C, C++, Java 语言的数据
   - 训练集：验证集：测试集 = 6:2:2 
2. Cross-Language Binary-Source Code Matching
   - 基于CLCDSA数据集创建了一个新的数据集
   - 以不同的编译器、不同的优化等级、不同的架构进行编译
3. Dataset for Pre-Training
   - 使用 `Clang` 以 `-O0` 的优化等级


## 5.3. Baselines
1. Cross-Language Binary-Source Code Matching
   - BinPro, B2SFinder, XLIR(LSTM)
2. Cross-Language Source-Source Code Matching
   - LLICCA, XLIR(LSTM)


## 5.4. Evaluation Metrics
- Precision, Recall, F1-score


# 6. EXPERIMENTAL RESULTS AND ANALYSIS
## 6.1. Effectiveness of IR for Cross-Language Binary-Source Code Matching
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-18.png>)
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-19.png>)
- 实验结果
  - 不同模型比较
    - XLIR 的结果表示，它能够实现二进制代码和不同编程语言的源代码的匹配
  - XLIR 模型自比较
    - Transformer-Based 的表现远大于 LSTM-Based 
- 结果分析
  - 作者认为这一效果源于转换成 LLVM-IR

## 6.2. Effectiveness of IR for Single-Language Binary-Source Code Matching
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-20.png>)
- 即使在单语言的场景下，XLIR依然能取得显著的优势

## 6.3. Extended Evaluation on Cross-Language SourceSource Code Matching
- 在相似阈值为 80%， XLIR 的性能大幅超过 LICCA
- XLIR 也具有混源语言的识别能力 

## 6.4. Influence of Major Factors
1. 预训练的作用
   - ![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-20.png>)   
   - ![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-21.png>)
   - 两者相比可以看出，预训练可以提升模型性能
     - small but perceptible margin
2. Transformer Encoder 的贡献
   - 文章所提供的所有实验结果显示，TRANSFORMER 的结果都优于 LSTM
   - 可能的原因：LLVM-IR都是长序列数据，而 TRANSFORMER 更擅长处理长序列数据
3. 编译选项
   - XLIR 面对不同的编译器、编译等级和架构，都展现出比 SOTA 更好的性能
4. 阈值
   - ![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-22.png>)
   - 阈值的变化对精确率和召回率有显著影响。当阈值增加时，精确率上升，但召回率下降。默认设置为0.8的阈值能够在精确率和召回率之间实现一个平衡，确保良好的检测效果。

# 7. DISCUSSION
## 7.1. Case Study
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-23.png>)
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-24.png>)
- 图中展示了具有相同语义的 C 语言的二进制代码件和 Java 语言的源代码，它们被转化成 LLVM-IR 后，具有相同的语义的片段
- 说明转换成 LLVM-IR 后，代码的原有的语义信息会被保留


## 7.2. Strength of XLIR
- 使用 LLVM-IR 提取信息
  - 这样可以保留更多的语义信息
- 端到端

## 7.3. Threats to Validity and Limitations
![alt text](<images/Cross-Language Binary-Source Code Matching with  Intermediate Representations/Cross-Language Binary-Source Code Matching with  Intermediate Representations-25.png>)
- 源代码需要是可编译的
  - 如果代码不完整或者语法错误等等，导致无法编译，就会进而导致无法生成相应的 LLVM-IR，也就无法使用 XLIR
- 仅支持部分编程语言
  - XLIR只支持具有静态LLVM编译器的编程语言

# Question
- 总觉得文章的创新性欠缺。。大都是复用之前的工作到BCSD领域中
- 不同形式的 IR 会不会有更好的效果