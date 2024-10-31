| **Title** | FASER: Binary Code Similarity Search through the use of Intermediate Representations |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Josh Collyer, Tim Watson |
| **Institution** | Computer Science, Loughborough Universit  |
| **Venue** | CAMLIS |
| **Year** | 2024 |

# Abstract
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations.png>)
- Intermediate representation 可以被用于 BCSD 领域，并且它之前并没有收到很多的关注
- IR 具有两种特性
  - 跨架构：可以被使用在 ARM、X86等等
  - 显示编码了函数的语义：IR 将不同的汇编代码转换为一种统一的形式，并且保留了函数语义
- FASER 具体方法
  - 将长文本转换器与 IR 结合起来
- FASER 特点
  - **跨架构**搜索
  - 无需手动设定特征、**预训练**、动态分析
- 应用于两个场景
  - 函数搜索
  - 漏洞搜索


# 1. Introduction
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-1.png>)
- 相关工作
  - 几年前，NLP 相关的方法已被使用到了 BCSD 领域中
  - 最近，TRANSFORMER 也被使用到了 BCSD 领域中，并且使用了类似 BERT 的预训练
- 存在问题
  - OOV问题
    - 如果一个新的词汇输入（并不在训练集中出现过），模型可能无法准确识别
  - 现有方法
    - `LLVM - IR` 被使用在从二进制到源代码的匹配 （XLIR）
    - `VEX - IR` 被使用在漏洞搜索 （Penwy）
- 本文方法 （FASER）
  - 结合了长文本转换器 **LONGFORMER** 和 **ESIL**
    - LONGFORMER 是一种**用于处理长文本**的 TRANSFORMER 的模型
    - ESIL 是一种 IR，由 RADARE2 生成
      - 优点：相比于其他中间表示（如LLVM、VEX等），ESIL更紧凑，通常生成的字符串更短
  - 使用 IR 可以避免对每一种汇编语言进行单独处理，只需要**统一处理**
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-2.png>)
- 本文贡献点
  - **只使用标准化**的方式，将二进制转换被为 IR 函数的字符串
  - **跨架构**
  - **无需预训练**，直接使用二进制函数目标进行训练
  - **第一个**支持 **RISC - V** 搜索的


# 2. Methodology
## 2.1. Chosen Intermediate Representation
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-3.png>)
- 选择 ESIL 作为 IR 的形式，是因为 ESIL 的**紧凑性**
- 指令转换为 IR 后，`ESIL - IR` 的**长度会明显更短**

## 2.2. Dataset
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-4.png>)
- Dataset - 1 for Function Search
  - 组成部分
    - 包含七个开源项目
    - 跨架构：ARM, MIPS, x86, etc..
    - 跨编译器：GCC, Clang
    - 跨优化等级
  - 目的
    - 进行函数匹配
    - 作者选取了最难的任务，即在测试中任意选择架构、编译器和优化登记进行排列组合
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-5.png>)
- Dataset - 2 for Vulnerability Search
  - 组成部分
    - 跨架构：ARM, MIPS, x86(作者基于原始数据集生成), **RISC-V**, etc..
  - 目的
    - 测试跨架构迁移能力：**RISC - V 并未出现在训练集中**，作者在测试集中加入了这个全新的架构，用于测试模型是否具有迁移能力


## 2.3. Data Generation
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-6.png>)
- 工具
  - bin2ml
- 过程
  - 1. bin2ml 使用 radare2 将二进制文件转换成 ESIL
  - 2. 将 ESIL 连接成为长字符串
  - 3. 截断过长的字符串
    - 可能会导致关键信息丢失；字符串的长度可选
  - 4. 标准化
  - 5. 删除重复数据

## 2.4. Normalization
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-7.png>)
- 目的
  - 词汇表大小可控：通过标准化大大减少变量的数量
  - 确保所有输入可以被编码
- 相关工作
  - 类似的方法**已经被使用**在了其他文献中
- 具体步骤
  - 立即数替换
    - 方法：将 `0xfffff` 开头或者长度为一到三个字符的十六进制视为立即数，并被替换为 `IMM`
  - 内存地址替换
    - 方法：将`0x`开头且后面有四个或者更多十六进制的被视为内存地址，并被替换为 `MEM`
  - 简化函数调用和数据调用
    - 关于函数调用的指令被替换成 `FUNC`
    - 关于数据访问的指令被替换成 `DATA`
  - 通用寄存器的替换
    - 寄存器更具它的大小被替换成相应的标记
    - 文章的实验部分包含了两组对照试验，对比寄存器替换与否对模型的影响


## 2.5. Depulication
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-8.png>)
- 目的
  - 同一源代码使用不同的编译器或者优化级别，可能会生成**相同的函数**，这些数据是**重复**且冗余的
  - 删除只出现过一次的函数
- 具体步骤
  - 1. 生成字符串
    - 每个经过标准化后的ESIL字符串**和它相对应的函数名连接在一起**，形成一个新的字符串
  - 2. 转换成 HASH
    - 将这些字符串转换成HASH
  - 3. HASH 比较
    - **通过 HASH 比较，确定出重复的函数**，每个 HASH 只保留一个
  - 4. 消除只出现一次的函数
    - 目的：去除那些没有变化且**没有学习价值**的函数
- 结果
  - 去重步骤减少了函数库中 20%-25% 的函数


## 2.6. Model Design
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-9.png>)
- LongFormer
  - 通过使用局部注意力和全局注意力机制解决 BERT 的自注意力机制的计算复杂度问题
- FASER 的结构: Input(ESIL STRING) >> LONGFORMER BLOCK >> DENSE LAYER >> OUTPUT(EMBEDDING)
  - 输入维度为 4096
  - 8 个 LongFormer Block，其中间维度为 2048，输出维度为 768
  - 2 个 DENSE LAYER：降维到 128，并输出
  - 局部注意力窗口大小为 512 个 toekn


## 2.7. Training Configuration
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-10.png>)
- FASER 只用于进行函数搜索任务，因此**不使用预训练**；并且，直接使用**度量学习**方式寻来呢 FASER
- 使用 Siamese 结构和 Circle Loss 损失函数
  - Circle Loss 能够更好的强调类间相似性的较大差异
  - 测试了 Cosine Embedding Loss 和 Triplet Loss，会导致训练不稳定，甚至模型崩溃
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-11.png>)
- 采样策略
  - 目的：保证在同一批次中有**正样本对**（同一函数的不同表示）
  - 方法：确保每个批次中相同函数至少有**m个样本**
- 动态创建正负样本对
  - 目的：针对模型的弱点进行训练
- 具体的训练方式
  - 训练集 DATASET - 1
  - 一个 epoch 为 100k 个函数，持续 18 个 epoch
  - ...


## 2.8. Comparison Approaches
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-12.png>)
- 函数搜索
  - Baseline 的选择：针对跨架构的模型
  - 单一架构模型不适用于本文跨架构的测试
- 漏洞搜索
  - 额外增加了 Trex 作为 Baseline


## 2.9. Evaluation Configuration && Merrics
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-13.png>)
- 任务
  - 都是给定一个函数，在目标池中搜索相似函数
- Function Search
  - 样本构成
    - 1 个正样本和 100 个负样本
  - 指标
    - Recall @1
    - MRR @10
- Vulnerability Search
  - 样本构成
    - 是 Function Search 的10倍大小（1k）
  - 指标
    - mean rank：平均排名
    - median rank：中位数排名


# 3. Evaluation
- NR M没有进行寄存器归一化的模型
- RN 进行了寄存器归一化的模型
## 3.1. Binary Function Similarity Search
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-14.png>)
- NRM 的性能明显优秀于其他模型

## 3.2. Binary Function Vulnerability Search
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-15.png>)
- 比例表示某模型在某架构下的四次实验的排名结果，排名越低性能越好
- NR 的性能明显优于其他模型

## 3.3. Zero Shot 
![alt text](<images/FASER/FASER: Binary Code Similarity Search through the  use of Intermediate Representations-16.png>)
- FASER 并不具有迁移能力

# Question
## 2.5. Deduplication
- 为什么需要删除只出现过一次的函数？训练集只出现过一次，并不代表真实任务中出现的频率很低

## 2.7. Training Configuration
- Circle Loss 强调类间相似性的较大差异，这个到底有多大性能的提升？是不是只是因为 Cos LOSS 和 三元损失无法收敛才使用的 Circle Loss

## 2.8. Comparison Approaches
- 模型选取的 Baseline 是不是年限过于久远了

## 3.1. Binary Function Similarity Search
- NRM 的绝对精度仍然不是很高

## 3.2. Binary Function Similarity Search
- 可以明显看出 NRM 的性能非常不稳定，甚至能到 122 名，是最差的
- 中位数排名和平均数排名这个指标是不是试验次数过于少了