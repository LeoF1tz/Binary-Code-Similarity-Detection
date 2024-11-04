| **Title** | CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques  |
|-----|-----|
| **Author** | Lichen Jia |
| **Institution** | University of Chinese Academy of Sciences |
| **Venue** | LCTES |
| **Year** | 2024 |
| **Paper** | [link](https://dl.acm.org/doi/10.1145/3652032.3657572#) |


# ABSTRACT
1. 函数内联影响
   - 不同的编译设置会导致内联的程度不同，进而导致同一源代码编译生成的二进制代码会出现显著差异
2. 代码提取技术
   - **与函数内联技术相反**，它可以识别并分离程序内的重复代码，将其替换为相应的函数调用
3. 本文方法 `CodeExtract`
   - 针对函数内联
   1. 使用代码提取技术，将函数内联引入的代码转换回函数调用
   2. 不能使用代码提取技术的部分代码，使用主动内联的方式将其嵌入到调用函数中
   - 有效消除函数内联的差异
4. 实验结果
   - 提升了 LB-BCSD 模型的精确性，整体幅度为 20%


# CONCLUSION
`CodeExtract` 基于**代码提取和主动内联**，有效解决了因函数内联带来的代码差异，LB-BCSD 的性能因此提升


# 1. INTRODUCTION
- 主要贡献点
  - 1. 研究函数内联对 LB-BCSD 模型的性能影响，解释现有方法无法解决函数内联的原因
  - 2. 函数内联问题转换为**计算重复代码**的问题，通过相似性匹配识别和提取二进制函数内部函数中的重复代码片段
    - `CodeExtract` 的方法
  - 3. 实验证明 `CodeExtract`  可以为 LB-BCSD 模型带来20% 的准确率提升
    - `CodeExtract`  的有效性


# 2. MITIGATING MEASURES FOR FUNCTION INLINING
- `BINGO` 和 `ASM2VEC` 内联模拟策略
  - 基于调用函数和被调用函数的指令条数比，和被调用指令的数量


## 2.1. Limitations in Inline Emulation
1. 不同的编译选项会导致不同的调用策略（**被调用函数会不同**）
   - 对于调用函数 `A` 使用不同的编译选项，会输出具有不同指令条数的二进制函数 `A'`
   - 指令越多的 `A'` 版本和指令数量较少的 `A'` 调用的函数数量会不一致
   - 当调用者函数的指令数量较多时，编译器可能会倾向于内联更多的被调用函数
   - `O3` 比 `O1` 的内联函数会更多
2. 现有的内联模拟策略仅针对**被调用函数**


# 3. Our Technique
1. 现有方法会受到**不同编译级别**的影响，`CodeExtract` 的目标是，**独立于优化级别**处理函数内联问题
2. `CodeExtract` 的技术
   1. 代码提取：如果程序中**存在重复代码**，代码提取技术将用于提取重复代码并将其转化为相应的函数调用
   2. 主动内联：如果**不存在重复代码**，则采用主动内联技术，将仅被调用一次的被调用函数内联到调用者函数中


## 3.1. When Does Function Inlining Introduce Duplicate Code?
- 基于**被调用函数的调用次数**分析在何种情况下函数内联会导致**重复代码**的产生


### 3.1.1 Callee Function Called Only Once
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img.png>)
1. `A` 调用，并内联 `C`，**且只有 `C2` 和 `C3` 被内联**，形成 `A'`
   - `C1` 和 `C4` 主要的作用是负责寄存器的保存和回复
2. 因为 **`C` 只被调用一次**，编译器**通常会删除原始的 `C`** 以减少代码量
   1. 只调用一次不适用于代码提取，使用**主动内联方式**应对
   2. 不同的编译条件会导致 `C` 可能会发生内联或者不内联，**主动内联可以消除内联与否的差异**


### 3.1.2 Callee Function Called More Than Once.
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-1.png>)
1. `A` 和 `B` 都调用，且内联了 `C`
   - 使用**代码提取**处理重复情况
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-2.png>)
1. `A` 和 `B` 都调用，且内联了 `C`，但是**不产生重复代码**，被称为**内联敏感函数**
   - `C` 包含分支指令，`A` 调用为真的分支，即 `C2` 和 `C3`；`B` 调用为假的分支，即 `C4`和 `C5`


## 3.2. Feasibility Argument for the Proposed Approaches
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-3.png>)
- 1. 表参数含义
  - 1. `nums` 被内联多次的函数数量
  - 2. `Inlining-Stable nums` 表示稳定内联的函数数量
  - 3. `D.R.` 表示内联函数占比，也反应了**内联敏感函数的比例** 
- 2. 结论
  - 92% 的函数是内联稳定的，**只有 8% 是内联敏感的**，即主动内联与代码提取的组合方法在消除函数内联引入的差异上具备良好的可行性和实用性


# 4. DESIGN
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-4.png>)
`CodeExtract` 输入是函数，输出是标准化后的额函数


## 4.1. Challenge
1. 识别重复代码的核心思想
   - 计算基本块建的相似性
2. 挑战
   1. 大量基本块的相似性计算开销
   2. 内联引起的指令差异：编译器通常会对被调用函数的指令进行优化，即使同一基本块，在不同的上下文中也会因为编译优化而有所差异，这样就会导致**相同基本块在不同的调用点的相似性降低**
3. 解决方法
   1. 过滤模块：只对关键块进行相似性计算
   2. 基本块标准化

## 4.2. Filter Module
1. 目的
   - 过滤非内联的基本块，以减少计算开销
2. 核心思想
   1. 识别**入口基本块**
   2. 只有**入口基本块及其子节点**可能是由内联引入的基本块
3. 入口基本块的特征
   - 当一个函数被内联到另一个函数中时，**内联后的所有基本块会在目标函数中从一个统一的起点**（即入口基本块）开始
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-5.png>)
4. 主函数：返回所**有识别出的入口基本块**
   1. 从函数集 `Functions` 中遍历每个函数 `func`
   2. 从 `func` 中遍历每个基本块`bb` 
   3. 调用 `isEntryBB` 判断该基本块是否符合**入口基本块**的特征
   4. 返回包含所有识别出的入口基本块的集合 `entryBB`
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-6.png>)
5. `isEntryBB`：判断该 `bb` 是否为入口基本块
   1. 枚举所有控制流程子图，限制子图大小为 `k`
      - 本算法中 `k=3`，即少于 3 个基本块的函数
      - 1. 减少计算开销
      - 2. 少量内联块的影响小，可以被忽略
   2. 对于每个子图 `subgraph` 中的节点 `node`
      1. 检查 `bb` 是否在 `node` 的 `ancestor` 中，即**祖先节点**
      2. 检查 `node` 的 `predecessors`，即**前驱节点**，是否在子图 `subgraph` 中
      3. 不满足 1 和 2 则视该 `bb` 为**入口基本块**
6. 前驱节点和祖先节点（直接和间接）
   1.  假设在控制流图中有以下路径：`A -> B -> C -> D`，i.e., 对于节点 `D`
   2.  `C` 为前驱，因为 `C` **直接指向** `D`（直接关系）
   3.  `A, B, C` 为祖先，因为它们都有**一条或者多条路径**到达 `D`（间接关系）


## 4.3. Similarity Calculate Module
1. 目的
   - 计算出过滤后的基本块间的相似性
2. 核心思想
   - 在同一二进制程序内，**被内联的被调用函数的控制流结构和指令通常保持不变**
   - 基于编辑距离的相似性计算方法，即将**基本块的指令视为字符串，通过计算这些指令字符串的编辑距离**来评估相似性
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-7.png>)
3. 特殊情况
   - 寄存器名称变化 (a) (b)
     1. 在函数内联过程中，**编译器优化可能会改变寄存器名称**，从而增加两个相似基本块之间的编辑距离
     2. 同一个基本块被内联到不同的调用点，**使用了不同的寄存器**
     3. 解决方法：对寄存器进行重命名，即对具有相同 `def-use` 行为的寄存器赋予**相同名称**
     4. i.e., 
        - (a) (b) 虽然指令中的操作数不一样，但是它们所实现的功能是类似的，故对它们进行重命名
   - 跳转指令目标 (c)
     - 统一替换为 `IMM` 标签 
     - i.e., 
       - (c) 中 `jle loc_40CDD5` 被重命名为 `jle IMM`


## 4.4. Repeated Code Extract Module
1. 目的
   - 识别函数 `f` 中的重复代码，并将其提取出来生成新的函数 `f`
2. 具体算法
   1. 输入 
      1. `f` 目标函数
      2. `fbbs` 目标函数中的相似基本块集合
   2. 输出
      - 提取出的函数 `f`
   - ![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-8.png>)
   - 对于函数 `f` 中的每个相似基本块 `tbb`，找到与之相似的其他基本块集合 `sbbs`
   - 对 `sbbs` 中的每个 `sbb`，调用 `findRepeatedCode` 函数来确定 `tbb` 和 `sbb` 之间的重复代码片段
   - ![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-9.png>)
   - `findRepeatedCode` 用于**递归地查找 `tbb` 和 `sbb` 之间重复代码**
   - 向前匹配
     - 从 `tbb` 和 `sbb` 的**前驱节点开始**，检查两者地相似性是否超过**阈值**。超过，则将该前驱节点添加到 `repeatedCode`，并继续向前检查
   - 向后匹配
     - 从 `tbb` 和 `sbb` 的**后继节点**开始进行相似性检查，直到相似性**不再满足阈值 `γ`**
   - 判断重复代码片段大小
     - 如果 `repeatedCode` 的大小小于阈值，则返回空集 `[]`
     - `repeatedCode` 的大小代表基本块的大小，**忽略基本块数量过小的重复代码段**    


# 5. EVALUATION
## 5.2. RQ1: Impact of Function Inlining on Model Accuracy
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-10.png>)
1. 实验方法
   - 开启/关闭函数内联 
2. 实验结果
   1. 没有函数内联：模型准确率较高，可以准确的捕捉到语义的信息
   2. 有函数内联：准确率显著下降，**指令间的关系复杂化**
3. 结论
   - 内联导致 LB-BCSD 模型的**准确性大幅下降**，平均下降30% 


## 5.3. RQ2: Impact of Callee Function Size on LB-BCSD Model Accuracy
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-11.png>)
1. 实验方法
   - 当编译器决定内联某函数时，首先判断该函数的基本块数量，只有在数量满足条件（<= x）时，才将该函数内联
2. 实验结果
   1. 基本块数量为 2 或更少：LB-BCSD 模型的**准确性仅下降了 3%**
   2. 基本块数量为 3 或更多：3 时，模型准确性下降了 **15%**；并且，随着基本块数量增加，模型的准确率显著降低
3. 结论
   - **被调用函数的复杂性（以基本块数量衡量）对 LB-BCSD 模型的准确性影响较大**，呈正相关


## 5.4. RQ3: Accuracy Enhancements Brought to LB-BCSD Models by CodeExtract
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-12.png>)
1. 实验方法
   - 编译时，按照内联模拟的规则进行内联
2. 实验结果
   1. 内联模拟的提升：9 - 12%
   2. 代码提取的提升：18 - 23%


## 5.5. RQ4: False Positive and False Negative Analysis
![alt text](<images/2024_LCTES_CodeExtract: Enhancing Binary Code Similarity Detection with Code Extraction Techniques/img-13.png>)
1. 实验方法
   - **`O3` 编译等级，在基本块数量为 3** 以上的函数中的假阳性率和假阴性率
2. 实验结果
   1. 基本块数 < 3
      1. 假阳性率增加：`CodeExtract` 主动内联仅调用一次的函数
      2. 假阴性率增加：`CodeExtract` 仅提取基本块数量 >= 3 的函数
   2. 基本块数 >= 3
      `CodeExtract` 的假阳性率和假阴性率显著降低  