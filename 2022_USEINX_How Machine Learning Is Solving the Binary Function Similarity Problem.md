| **Title** | How Machine Learning Is Solving the Binary Function Similarity Problem |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Andrea Marcelli |
| **Institution** | Cisco Systems, Inc.  |
| **Venue** | USENIX |
| **Year** | 2022 |

# 1. Introduction
![alt text](<images/How ML Is Solving the BCSD/img.png>)
![alt text](<images/How ML Is Solving the BCSD/img-1.png>)
![alt text](<images/How ML Is Solving the BCSD/img-2.png>)
![alt text](<images/How ML Is Solving the BCSD/img-3.png>)
- 问题定义
  - 输入：一对二进制函数
  - 输出：两者的相似性分数
- 难以检测的原因
  - 不同的架构、工具链条、编译等级
- BCSD 的重要性/应用
  - 逆向工程
    - 找到无符号数
  - 漏洞检测
    - 定位已知漏洞函数
    - 漏洞分类
- 当前存在诸多没有解决的问题


## 1.1. Challenge
![alt text](<images/How ML Is Solving the BCSD/img-4.png>)
1. 研究结果的**不可复现性**
   - 诸多原因导致很多论文不可复现
   - 论文中的实验比较方式因此受限，仅能和早期的几种经典技术比较，这些技术甚至可能并不是用来解决该论文的方向的问题
     - Gemini 和 Genius 等等技术被反复提及
   - 这点在机器学习领域非常常见
2. **评估结果通常不透明**
   - 不同方法的目标、设置和粒度不尽相同，实验的数据集也各不相同，导致即使是最基本的评估数据也难以进行直接比较。论文中的很多细节缺失，使得即使使用相同的数据集，也难以准确地复现其实验流程
3. 研究领域的**高度碎片化**
   - 不同的研究路线各自独立且缺乏交互，导致难以明确哪种方法在何种情况下最有效。论文中常常为了证明方法的创新性而采用更复杂的技术组合，这些排列组合是否真的对性能产生正向影响

## 1.2. Contributions
![alt text](<images/How ML Is Solving the BCSD/img-5.png>)
![alt text](<images/How ML Is Solving the BCSD/img-6.png>)
- 进行了**首次系统性的测量研究**，特别是机器学习领域
- 为了确保结果的公平性，在统一的框架下进行了实现，采用了相同的特征提取方式，避免不同工具链导致的差异
- **基于图神经网络的机器学习模型**在大多数任务中表现出色，同时推理速度也非常快
- 尽管一些论文宣称超越了现有技术，但在相同数据集上的测试结果却显示其准确率非常接近 
  - 这个就跟之前提到的，不同任务、不同数据集、不同指标等等原因导致的


# 2. The Binary Function Similarity Problem
![alt text](<images/How ML Is Solving the BCSD/img-7.png>)
- 本文中的相似函数的定义为
  - 即使编译环境（如编译器、架构）不同，只要函数源代码相同，就认为它们是相似的


## 2.1. Measuring Function Similarity
### 2.1.1. Direct comparison
![alt text](<images/How ML Is Solving the BCSD/img-9.png>)
- 方法
  - 对成对的函数进行逐一比较。研究人员通过特征提取技术，从二进制函数中提取出有意义的特征，然后使用机器学习模型来判断这些特征是否表示相似的函数
- 缺点
  - **需要遍历整个数据集**，与每一个候选函数进行比较。也就无法被应用在大规模搜索中
- 创新点
  - 检索方式
- ~~不常用，本文关于这部分所引用的文献最新不超过2019年~~

### 2.1.2. Indirect comparison
![alt text](<images/How ML Is Solving the BCSD/img-8.png>)
- 方法
  - 将输入特征映射到**低维表示**中，再用**距离度量**进行比较（欧几里得距离/**余弦距离**）
- 特点
  - 高效，**允许进行一对多的比较**：将数据集中的每个函数都映射到低维表示，然后，当新函数需要进行匹配时，只需要将新函数转换成低维表示再与这些已转换的函数进行比较即可


### 2.1.3. Fuzzy hashes and embeddings
![alt text](<images/How ML Is Solving the BCSD/img-10.png>)
- 模糊 HASH
  - 将相似的输入映射成相似的 HASH 
- ~~不常用~~


### 2.1.4. Code Embedding
![alt text](<images/How ML Is Solving the BCSD/img-11.png>)
- 方法
  - 将汇编代码视为文本，并采用 NLP 技术
  - 为每一个标记或指令生成一个嵌入表示
- 三种模型
  - 基于 word2vec：将指令视为词语，通过学习不同指令的上下文关系生成嵌入表示
  - 基于 seq2seq：将不同架构的代码映射到相同的嵌入空间
  - 基于 **BERT或TRANSFORMER**：最新的 NLP 技术
- 缺点
  - OOV问题
    - 实际使用场景中可能会出现训练集中从未出现过的指令
  - 模型的输入大小也会收到限制
    - 过大可能会被截断
- 这些缺点导致不同模型的**嵌入等级**不同
  - 函数级
  - 指令级
  - 基本块级
  - 这里指的是针对什么规模的数据进行嵌入


### 2.1.5. Graph Embedding
![alt text](<images/How ML Is Solving the BCSD/img-12.png>)
- 方法
  - 使用GNN 等技术基于控制流程图计算图的嵌入向量


## 2.2. Function Representations
### 2.2.1. Raw bytes
![alt text](<images/How ML Is Solving the BCSD/img-13.png>)
1. 直接对二进制信息进行编码
2. 将二进制原始字节和其他信息结合，例如 `CFG`


### 2.2.2. Assembly
![alt text](<images/How ML Is Solving the BCSD/img-14.png>)
- 可以对汇编指令进行编码
  - 操作数、操作符等等


### 2.2.3. Normalized assembly
- 原因
  - 汇编指令中的内存地址、操作数等等存在非常多不同的常量值
- 方法
  - 将这些常量值使用某些特定的标记代替，`MEM`, `NUM`, etc..
- 效果
  - 减少词汇表大小


### 2.2.4. Intermediate representations
![alt text](<images/How ML Is Solving the BCSD/img-15.png>)
- 特点
  - 1. 语法无关
  - 2. 架构无关
  - 3. 代码结构可以得到简化


### 2.2.5. Structure
- 方法
  - 通过分析函数的内部结构和其在程序中的角色进行相似性分析
- 种类
  - CFG, ACFG, 寄存器图, 函数调用图,etc..


### 2.2.6. Data flow analysis
- 为什么需要数据流分析
  - **具有相同语义的函数可以可能会由不同的指令组合实现**，因此，单纯以代码结构分析相似性可能不够准确
- 主要方法
  - 程序切片
    - 通过分析程序中变量的定义和使用关系，来提取与特定变量或表达式相关的代码片段的方法
  - 数据流边
    - 将控制流图中的块之间的数据流边视为附加特征，来增强函数行为的表示


### 2.2.7. Dynamic analysis
- 方法
  - 通过实际执行代码来观察其行为和特征的方法，它能捕捉代码在运行时的实际表现
- 具体
  - 输入输出关系的特征提取
  - 从执行轨迹中提取与体特征
  - 仿真和混合技术


### 2.2.8. Symbolic execution and analysis
- 方法
  - 将输入表示为符号变量，从而探索程序中所有的可能路径


# 3. Selected Approaches
- 本文的测试是在统一的数据集上
- BCSD 领域中大多数论文只是相同技术的微小变化


## 3.1. Selection Criteria
- 我觉得这里的选择方式可以被视为优秀方法的基准
  - 可以在大数据集中使用
  - 速度快


## 3.2. Selected Approaches
![alt text](<images/How ML Is Solving the BCSD/img-16.png>)



# 4. EVALUATION
## 4.2. Dataset
- Dataset中包含了多种编译器、架构、优化等级生成的不同的二进制文件，并且，简单的函数被过滤
- 数据集可用性
  - 向公众开放了数据集及其相关的脚本和补丁


## 4.3. Experimental Settings
- XM 任务最为复杂，函数对来自**任意架构、位宽、编译器、编译器版本和优化选项**
  - 按函数包含的基本块数量分为三种子数据集：XM-S, XM-M, XM-L
- 评价指标
  - AUC
  - MRR
  - Recall@K


## 4.4. Fuzzy-hashing Comparison
- ~~不如 NLP 方法~~

## 4.5. Machine-learning Models Comparison
- 模型比较的挑战
  - 在神经网络中，多个因素可能影响结果，本文使用**网格搜索**方法确定每个模型的最优的超参数
- 模型结果
  - GNN 的结果是最好的
  - 基于机器学习的方法在 AUC 指标上的结果相近，区别在于 MRR 和 RECALL@K
  - 某些模型适合在特定条件下执行任务


### 4.5.1. TREX
- Trex 的结果表示 Transformer 在处理跨架构、跨语言的任务中具有优势


### 4.5.2. GNN
- 在 GNN 的变种模型中，Li 所提出的 GNN 模型要优秀于同为 GNN 模型的 Gemini，体现在模型精度方面
- 在执行时间方面，并没有优势

### 4.5.3. 特征数量的影响
![alt text](<images/How ML Is Solving the BCSD/img-17.png>)
- 特征复杂度排序
  - BoW >> manual >> Nofeatures
- 特征的数量和模型的性能并不成正相关
- 特征的数量增加，会导致计算复杂度增加，进而导致模型训练时间的增加

### 4.5.4. Modelling functions using a textual encoder
- 基于指令嵌入的句子编码器
    - 在小函数池的效果比较好
      - 这个我觉得应该是跟 OOV 有关，小函数池的出现的绝对函数数量比较少，也就可以认为 OOV 问题会比较少
    - OOV 问题在 x86/64 最为严重，因为 CISC
    - 随着指令长度的增加，SAFE 的性能也增加
      - 可能是指令长度增加，模型能看到更多的词，OOV更少？

### 4.5.5. Asm2Vec and other paragraph2vec models
- Asm2Vec 和它的两种变体性能上并没有差异
- 针对单一架构的方法与跨架构的方法相比，在特定架构中并没有明显的性能优势
- 编译变量越多，AUC 越低
  - 场景越复杂，模型的性能越差

### 4.5.6. Comparing efficiency
- SAFE 最快


# 5. Discussion
## 5.1. The main contributions of machine-learning
- ML 模型可以捕捉到更复杂的特征，这是模糊哈希无法实现的
- ML 模型可以更好的适应**多个编译变量**的场景

## 5.2. Which is the role of different sets of features?
- Zeek 表明了数据流信息能够很好的提升模型性能
- 模型的选择、损失函数对最终结果的影响权重占比很大
- 基本块特征 （例如ACFG）的效果很好

## 5.3. Do different approaches work better at different tasks? In particular, is the cross-architecture comparison more difficult than working with a single architecture?
-  大多数机器学习模型，无论是处理相同架构还是跨架构任务，在所有评估任务中的表现相似。这表明，它们具有一定的通用性，可以应对多种复杂任务，尤其是在无需专门训练的情况下，使用通用数据（XM）即可获得接近最佳的结果

## 5.4. Is there any specific line of research that looks more promising as a future direction for designing new techniques?
-  结合GNN和汇编指令编码器的模型是未来的一个有前景的方向
-  训练策略和损失函数对模型性能有显著影响