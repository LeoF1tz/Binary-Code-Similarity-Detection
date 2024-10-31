| **Title** | Code is not Natural Language: Unlock the Power of Semantics-Oriented Graph Representation for Binary Code Similarity Detection  |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Haojie He |
| **Institution** | Shanghai Jiao Tong University  |
| **Venue** | USENIX |
| **Year** | 2024 |

# Abstract
![alt text](<images/Code Is Not Natural Language/img.png>)
- 现有方法的局限性
  - 基于指令流：将二进制代码视为自然语言，忽略了明确定义的语义结构
  - 基于 CFG ：仅控制流信息得到了应用，其他信息被忽略
- 本文核心观点
  - 二进制代码具有其明确定义的语义结构
    - **指令内部结构**：每条指令中操作数和操作符的关系明确
    - **指令间的关系**：数据流、控制了和执行顺序限制
    - **隐式约定**：例如函数调用约定，使用哪些寄存器传递参数或者返回值
- 本文方法
  - 一种**面向语义的图形表示**作为输入给深度神经网络
    - 这种表示带有完整的语义信息
  - 一种**轻量级**的**多头** softmax 用于**融合多种信息**
- 实验结果
  - 本文提出的方法明显优于 SOTA 

# Conclusion


# 1. Introduction
![alt text](<images/Code Is Not Natural Language/img-1.png>)
- BCSD 的作用
  - 检索头龙函数
  - 识别恶意软件的代码片段
  - ...
![alt text](<images/Code Is Not Natural Language/img-2.png>)
- 现有挑战（两者刚好相反）
  - 语义相同，但是语法表示完全不同；尤其体现在同一源代码被编译成不同架构的二进制代码中
  - 语法相似，但是语义完全不同
- 现有方法
  - 基于 CFG
    - ![alt text](<images/Code Is Not Natural Language/img-6.png>)
    - 具体方法
      - 以 CFG 作为输入，利用 GNN 提取特征
      - 近年来也结合了关于 NLP 的技术
    - 发展
      - 相比之下，有的方式会尝试结合更多的语义信息，比如数据流
  - 基于指令流
    - 具体方法
      - ![alt text](<images/Code Is Not Natural Language/img-4.png>)
      - ![alt text](<images/Code Is Not Natural Language/img-5.png>)
      - 将二进制代码视为自然语言，并引入自然语言处理（NLP）技术
        - LSTM，自注意力机制，预训练
        - 这些都是 NLP 相关的
    - 缺点
      - 预训练代价昂贵
      - **没有充分利用代码的语义特征**
![alt text](<images/Code Is Not Natural Language/img-7.png>)
- 二进制代码和自然语言的区别
  - 1. 二进制代码具有**明确的结构、语义和规定**
  - 2. 指令可以重新排序
    - 尤其体现在编译器会在优化代码的过程中对指令进行重新排序
  - 3. 指令某些部分可能与语义无关
![alt text](<images/Code Is Not Natural Language/img-8.png>)
- 基于二进制代码并不能完全视为自然语言，本文的见解（如何更好的捕捉二进制代码的语义）
  - 1. 揭示指令内部结构，显示操作数如何被操作符使用
  - 2. 揭示指令间的关系，如控制流和必要的执行顺序
  - 3. 排除与语义无关的元素，例如用于临时存储数据的寄存器以及不必要的执行顺序限制
  - 4. 编码其他隐式知识，如调用约定
![alt text](<images/Code Is Not Natural Language/img-9.png>)
- 本文方法
  - 1. 二进制函数转换为 SOG
  - 2. 使用 GNN 聚合每个节点的邻域信息
  - 3. 使用多头 Softmax 生成图嵌入向量
  - 4. 基于嵌入向量计算相似性分数
![alt text](<images/Code Is Not Natural Language/img-10.png>)
- 贡献点
  - 1. 二进制代码表示方法，SOG
    - 保留完整的语义信息
    - 剔除语义无关信息
  - 2. 多头 Softmax 融合器
    - 能够融合 SOG 的多个方面的信息
  - 3. 用于解决 BCSD 的 HermesSim
  - 4. 通过实验证明 SOG 的性能优于 SOTA


# 2. Background and Motivation
## 2.1. A Toy IR
![alt text](<images/Code Is Not Natural Language/img-11.png>)
![alt text](<images/Code Is Not Natural Language/img-12.png>)
-  从 toy IR 中，可以看到具有各自特点的四类符号
   -  寄存器：`ri`
   -  操作符：全大写，且只出现在指令的等号左侧或者右侧第一个
   -  整数字面量：由纯数字构成，十六进制
   -  标签：`Li`，一般出现在函数的开头，标记基本块的开始，或者函数的结尾，表示函数返回


## 2.2. Semantically Equivalent Variants
![alt text](<images/Code Is Not Natural Language/img-15.png>)
![alt text](<images/Code Is Not Natural Language/img-13.png>)
- 基于 NLP 方法的处理是**将句子视为一个由多个 TOKEN 组合而成的长序列**，每个 TOKEN 包含两个方面的信息，**这两个信息的结合**为每个 TOKEN 的实际输入
  - 内容信息
  - 位置信息
  - 以 `r0` 为例，它的内容信息是自身，即`r0`；位置信息是`0`，表示在该 Sequence 中为第一个 TOEKN
- 修改内容或者位置都会导致这个输入的变化，就算该指令的语义并不发生改变，模型也需要重新学习
- 举出三个**不改变语义但是信息发生变化**的例子，模型需要去学习到这些改变并不影响指令的语义
  - ![alt text](<images/Code Is Not Natural Language/img-14.png>)
  - (b) 表示两个指令的顺序发生变化，即位置信息变化
  - (c) 表示将整个部分的指令挪到其他位置，即位置信息变化
  - (d) 表示将数据存储到另外一个寄存器中，即内容信息变化
    - 这里表现出 `def-use` 关系构成了句子的语义实体，即**寄存器的具体选择和存储的值是无关的**
- NLP 可以通过大量数据去学习到这些语义并不发生变化，但是，代价是非常昂贵的，需要数据和更多的训练

## 2.3. Implicit Structures of Binary Code
![alt text](<images/Code Is Not Natural Language/img-16.png>)
- 二进制代码的语义
  - 1. 指令内部结构：操作符和操作数之间存在明确关系
    - `ADD` 表示指令的操作是什么
    - `r1, r2, r3` 表示指令执行的目标地址
  - 2. 指令间的关系
    - 数据关系：使用了别的指令定义的变量
    - 控制关系：表示基本块级别的控制流
    - 效果关系：表示指令执行的顺序
      - 引入指令执行关系的必要性：随意更改两条指令的执行顺序可能会导致指令执行结果的更改
      - 并且，只考虑数据和控制关系并不能代表完整的语义
  - 3. 函数级别的规则
    - 调用关系等，也算作是二进制代码语义的一部分
![alt text](<images/Code Is Not Natural Language/img-17.png>)
- 指令之间的执行顺序是相对比较灵活的，只需要**引入必要**的执行顺序，以避免过多的限制
- 将指令的必要执行顺序转换成对**潜在的数据流**
  - 例如 `STORE` 指令修改了一个寄存器的内容，`CALL` 指令对这个寄存器进行读或者写，两者就形成了**潜在的数据流**
  - 两条指令对**同一个内存地址**先后进行**写读或者写写**
- 表示这种**潜在数据流**的方法
  - 使用**抽象的临时变量表示特定的内存区域**，任何对该内存区域实现读/写的指令都被认为使用了该变量



## 2.4. Step to Semantics-Oriented Graph
![alt text](<images/Code Is Not Natural Language/img-18.png>)
- 选择图作为结构的原因
  - 指令顺序执行的方式是由机器的特点所限制的，而不是因为语义导致的顺序执行
![alt text](<images/Code Is Not Natural Language/img-19.png>)
- (a) to (b) 基于**指令**的完全语义图
  - 目的
    - 将指令序列提升为图表示，揭示指令间的关系
  - 方式
    - 基于数据流图，指令为节点，指令间关系为边
  - 特点
    - 在满足指令间关系的情况下，随意更改指令的执行关系并不改变语义
- (b) to (c) 基于**Token**的完全语义图
  - 目的
    - 分解指令，揭示指令内部的结构
  - 方式
    - 将指令分解成单个Token
  - 特点
    - 指令内部关系可以被**理解为使用**
      - `r0 = LOAD 0x1000`
      - `LOAD` 使用 `0x1000`
      - `r0` 使用 `LOAD 0x1000` 的结果
  - 结果
    - 使得指令的**控制流和数据流被分离**，更好的为 (d) 中剥离临时存储空间服务
![alt text](<images/Code Is Not Natural Language/img-20.png>)
- (c) to (d) **SOG**
  - 方式
    - 进一步化简关于临时存储的 TOKEN
    - 只用做临时存储的区域被省略，直接连接它们的输入和输出输出
    - 因未初始化的存储区域携带了语义信息，故被保留
      - 未初始化的存储区域通常是函数调用返回的结果
  - 目的
    - 消除与语义无关的元素：寄存器或栈槽的选择对语义无关，SOG 可以移除这些无关的节点
    - SOG 重点建模指令之间的语义关系，而不是关注细节上不重要的硬件特性


# 3. Details of SOG and its Construction
![alt text](<images/Code Is Not Natural Language/img-27.png>)
1. 控制子图：**构建节点**
   - 1. 每个基本块以 `BR` 分支指令**结束**
   - 2. 控制流边由**目标节点指向节点**
     - 有点反直觉，一般箭头指向的是执行顺序
   - 3. 对于具有多个后继的分支，使用 `PROJ` 节点处理不同的后续路径
     - `SOG` 的核心思想是一个节点只干一件事情
2. 数据子图：基于 `def-use` 关系构建，**构建边**
   - 1. 一般情况
     - 每个指令的**操作符会成为一个节点**，每个操作数都通过一条有**向边指向定义该操作数的前一个节点**
   - 2. 未定义的操作数节点
     - 创建一个**新的节点**来表示这些未定义或初始值
     - i.e., `mov eax, 0xAAA` 中，会为 `0xAAA` 创建一个新节点
   - 3. 多重定义：引入 `PIECE` 节点
     - 将多个定义的值抽象地**拼接在一起**，从而形成一个唯一的值
     - i.e., `mov eax, 0xAAA`, `mov al, 1`, `mov edx, eax`，第三条指令的结果依赖于前面两条指令，则**SOG 使用 PIECE 连接**
   - ~~不太理解这里的 PROJ（1,3）是什么含义~~
3. 效应子图
   - 1. 构建方式和数据子图一致
     - 理由是其可以被视为**操作码和操作数**
   - 2. 内存指令和 I/O 指令
4. PHI 节点
   - 目的：确保每个指令的唯一性和简洁性，使得每个变量**只有一个定义**（与 SSA 理念一致
   - 方式：当程序在不同分支中对同一变量进行赋值时，Phi 节点在分支的合并点，Phi 节点会根据进入该节点的控制流路径选择合适的值
   - i.e., PHI 会根据控制流选择使用 `0x1` 还是 `0x2`


# 4. BCSD with SOG
## 4.1. Framework
![alt text](<images/Code Is Not Natural Language/img-21.png>)
1. 函数嵌入
   1. SOG 转换：二进制函数被转换成 SOG 图
   2. 嵌入转换：经过标准化后，将**节点和边**转换为嵌入向量
   3. 输入 GGNN：GGNN 用于聚合每个节点的相邻节点信息
   4. 输入多头 Softmax 聚合器：聚合多个维度的信息，将图中所有节点汇总为一个整体图嵌入
2. 相似性计算
   - 孪生网络相似性计算：在向量空间中计算两个向量之间的相似性
3. 损失函数
   - 基于边距的损失函数
4. 负采样策略
   - 基于距离加权的负采样策略


## 4.2. Graph Normalization and Encoding
1. 节点作为**Token**映射为向量，其中**Token**被细分为三种类型
   1. 常量类型
      - 常见的单独记录词汇表，不常见的全部归为一类  
   2. 指令类型
      - 常见的单独记录词汇表，不常见的全部归为一类 
   3. 寄存器类型
      - 词汇表包含不同架构下的寄存器
2. 边带有**两个属性**，分别映射为向量，再相加得到最后的嵌入向量
   1. 作用属性（数据、控制和效应）
   2. 位置属性（对应操作数的位置） 


## 4.3. Local Structure Capture
GGNN 通过逐**层融合节点的输入和输出**关系来捕捉局部结构信息


## 4.4. Multi-head Softmax Aggregator
1. 作用
   - 允许模型在多个表示空间中学习不同的图嵌入
2. 步骤
   1. 特征转换和激活：先通过**线性层**将节点嵌入转换到一个新的表示空间，然后通过 **ReLU 激活层**选择性地保留特征
   2. 层归一化和 Softmax 聚合：层归一化可以平衡每个节点的特征值范围，确保特征的均衡分布。然后通过 Softmax 聚合器，模型可以为每个节点嵌入分配一个加权值
   3. 多头嵌入拼接：多头的输出嵌入拼接后，通过线性层生成最终的图嵌入


# 5. Evaluation
## 5.2. Comparative Experiments
![alt text](<images/Code Is Not Natural Language/img-22.png>)
![alt text](<images/Code Is Not Natural Language/img-23.png>)
1. 测试类型
   - `XA` 跨架构和位宽
   - `XO` 跨优化等级
   - `XC` 跨编译器、编译器版本和优化级别
   - `XM` 跨架构、位宽、编译器、编译器版本和优化等级
2. 结果
   1. 仅当函数池非常小时，`jTans` 的性能才能和 `HermesSim` 相当；其余时候都是 `HermesSim` 更好
   2. 参数量
     - `HermesSim` 的参数量在基于机器学习的 BCSD 方法中是最小的，而且其他模型至少大了一个数量级
   3. 随着函数池增加，其余模型会发生显著的性能下降，而 `HermesSim` 的性能下降较慢，表现出更高的鲁棒性和稳定性


## 5.3. Ablation Study
![alt text](<images/Code Is Not Natural Language/img-25.png>)
![alt text](<images/Code Is Not Natural Language/img-24.png>)
1. 实验目的
   - 使用**消融实验**探究 `SOG` 和 `多头 SOFTMAX 聚合器` 的性能
2. SOG
   - 与 `CFG` 和 `草根表示` 方法相比，`SOG` **显著提高了模型的 Recall 和MRR**
3. 多头 SOFTAMAX
   - 多头聚合器通过多个表示空间聚合节点特征，有效提升了模型在大规模检索任务中的表现
 

## 5.4. Runtime Efficiency
![alt text](<images/Code Is Not Natural Language/img-26.png>)
1. 与 `CFG`, `ISCG`, `DFG`, `CDFG` 相比，虽然推理时间不占优势，但是这些图在**属性提取上**更加复杂（数据流等信息）
2. 与 `TSCG` （语义完全图）相比，`SOG` 的节点数、边数显著降低，即在保证语义完整性的情况下，`SOG` 显著降低了图的复杂程度；另外，推理时间也得到大幅度的优化


# 6. Discussion
1. 脏效应问题
   - 指的是在**编译过程**中引入的一些**效应相关**指令可能会对语义分析产生干扰
   - 未来可以通过**加载-存储消除分析**来清理不必要的栈操作
2. I/O 效应模型
   - 对于直接与硬件交互的代码，I/O 效应模型可能更合适
3. 额外信息与编码能力的扩展
   - **字符串、整数和符号等额外信息**对于代码语义理解很有帮助，未来可以利用 NLP 方法将这些特征整合到 SOG 中，以丰富其语义表达
4. 控制流恢复失败的应对
   - 由于间接分支控制流恢复的局限，SOG 以及其他表示方法在这方面都有不完整性。可以尝试向模型提供引用数据等辅助信息，使模型能更好地推断控制流关系