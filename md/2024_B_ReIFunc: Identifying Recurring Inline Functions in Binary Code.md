| **Title** | ReIFunc: Identifying Recurring Inline Functions in Binary Code |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Huaijin Wang |
| **Institution** | Hong Kong University of Sience and Technology |
| **Venue** | TOSEM |
| **Year** | 2023 |

# ABSTRACT
1. 本文方法：子图重构+深度学习的方式识别内联函数
2. 实验结果：精确率超过 99%，召回率可接受

# CONCLUSION
同上


# 1. INTRODUCTION
1. 函数内联带来的挑战
   1. 内联函数和其调用函数间没有明确的边界
   2. 指令、CFG 等会发生显著变化
   3. 现有方法无法应对大数据集
2. 贡献点
   1. 使用子图同构和深度学习识别反复出现的内联函数的边界，并使其代码片段和函数名称关联
   2. 在有限的训练数据集下，保持对内联函数识别的高精度和可接受的召回率
   3. 开源


# 2. BAKGROUND AND MOTIVATING EXAMPLE
## 2.1. Function Recognition and Function Identification
1. 函数识别
   - 确定所有函数在二进制文件中的位置
2. 函数确认
   - 确认二进制文件中是否含有目标函数


## 2.2. Inline Function Identification
1. 函数内联的形式：两者的二进制表示中没有区别
   1. 源代码显式声明
   2. 编译器优化
2. 指纹识别方式在数据量大时，时间开销不可接受


## 2.3. A Motivating Example
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img-1.png>)
1. 图解
   - 函数 `solve` 两次调用 `doit`，在 `O3` 等级优化下，`doit` 被内联，即 `doit` 为重复内联函数
2. 难以区分内联函数的原因
   1. `solve` 的 CFG 中原代码和被内联代码**没有清晰的边界**
   2. 重复内联函数具有**不同的调用点，其拥有不同的上下文环境**


# 3. PROBLEM STATEMENT
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img.png>)
1. 观察 1：45% 的内联函数是**重复出现**的
2. 观察 2：语义相似的基本块（仅存在指令级别上的区别） 99% 都是与内联相关
3. 本文方法
   - 识别出语义相似的基本块，并与编译器内联的函数进行匹配
   - 形式化表达
     - 对于一个包含多个基本块 $BB$ 的二进制程序 $BIN (BB\in BIN$)
     - 本文目标是根据 $BIN$ 生成多个基本块集合 $RIF (BB\in RIF)$，这些 $RIF$ 在 $BIN$ 中反复出现，并且附带 $RIF$ 对应的函数名称 

# 4. REIFUNC
## 4.1. ReIFunc Overview
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img-2.png>)
1. Recurring Inline Function Recognition
   1. 基本块配对
   2. 子图重构
   3. 重写
2. Recurring Inline Function Identification 
   1. 编译
   2. 训练
   3. 生成
   4. 估计
   5. 比较

## 4.2. BB Pairing
1. 任务
   - 将具有相似语义的基本块分为一组
2. 挑战
   1. 指令集差异：具有相同语义，但是指令集不一致
      - `mov eax, 1` 和 `xor eax, eax; add eax, 1` 最终结果都是 `eax = 1`
   2. 寄存器和助记符的差异
      1. 相同语义的汇编代码可能使用了不同的寄存器
      2. 不同的助记符具有相同的语义 `movs` 和 `movsb`
   3. 指令顺序差异
      - 编译优化可能导致指令集的顺序发生变化
   4. 指令数量差异
      - 某些情况下，指令数量的减少不会导致语义发生变化
3. 解决方法
   1. 代码提升（标准化）
      - 将汇编代码提升为 `Microcode`，即 IR
   2. 指令编号
      - 对操作码和操作数进行**质数编号**，进而将**指令转换为质数乘法**
   3. 特征字典构建
      - 为指令编号创建一个字典，**使不同顺序的指令集在字典上保持一致**
   4. 相似性计算
      - 使用 `Jaccard` 系数计算语义相似性
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img-3.png>)
4. 具体算法
   - 输入：`F` 目标函数的 `IR`；输出：`D` 存储相似基本块的字典，`key` 为基本块索引，`value` 为与 `key` 相似的基本块索引
   - `L` 目标函数中的基本块编号，`E` 存储基本块对应的向量，`D` 存储相似基本块的字典，`S` 相似性阈值
   1. 计算特征向量
   2. 计算相似性并分组


## 4.3. Longest Equivalent Subgraph
1. CFG = $<V, E, L>$，即基本块，边和嵌入向量
2. 子图同构
   1. 边必须一一对应
   2. 基本块 $V$ 和其对应的特征向量，这两个维度的相似性必须满足阈值
3. 最长等价子图识别过程
   1. 遍历基本块配对字典 `D`：对于 `D` 中的每个相似基本块对 `(i,j)`，将其分别作为 `G1` 和 `G2` 的起始节点
   2. 初始化队列和路径：队列用来存储后继基本块对 `(i+1, j+1)`，路径用来存储**已匹配且相似**的基本块路径
   3. 不断从队列中取出基本块对，将他们的后继节点最为基本块对加入队列，判断其是否相似
   4. 如果路径长度大于阈值，则视为有效的等价子图


## 4.4. Rewriting
1. 目的
   - 将识别出的内联函数在**不对二进制文件的汇编代码进行直接修改**的情况下，进行重写，转换成适合模型的输入的格式
2. 方法
   - 在 IDA 生成的数据库文件中添加 `.inline`，以存储内联函数的汇编代码及其内存地址映射


## 4.5. Recurring Inline Functions Identification
1. 重写后的 RIF 代码输入到表示学习模型
2. 确定 RIF 来源的步骤
   1. 编译构建数据集：通过三个优化级别编译源代码，并**禁用函数内联**以确保所有函数保留在二进制文件中
   2. 训练得到向量表示：使用数据集对表示模型进行训练，得到每个函数的向量表示
   3. 基于向量表示为每个函数生成指纹库 
   4. 将 `ReIFunc` 识别出来的内联函数输入到模型中，生成对应的嵌入向量
   5. 与指纹库中的向量进行比较，判断该 RIF 函数的来源函数



# 5. EVALUATION
## 5.1. Performance in RIF recognition
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img-4.png>)
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img-5.png>)
1. 实验结果
   1. 精确率：在所有优化级别和编译器设置下，**精确率均高于 99%**
   2. 召回率：`ReIfunc` 的召回率较低，但跟现有技术比仍有竞争力
   3. 特征字典相比颜色的准确率显著提升
2. 实验分析
   - 颜色：**未能体现指令数量和操作数**，仅分类助记符类别
   - 特征字典：语义特征更多样
   - 极端例子：一个块包含 1000 条 `mov`，另一个块包含 1 条 `mov` 两者的颜色是相同的，而特征字典可以反映出更多信息


## 5.2. Performance in riff Identification
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img-6.png>)
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img-7.png>)
1. `Trex` + `ReIFunc` 的性能最好
2. 编译器会优化导致真实函数和查询函数的相似性大大降低
3. LES 的大小和 Recall@1 的指标呈负相关，影响因素：
   1. 编译器优化
   2. 上下文关系


## 5.3. Complexity Analysis
![alt text](<images/2024_B_ReIFunc: Identifying Recurring Inline Functions in Binary Code/img-8.png>)
1. 时间成本
   1. 
2. 指纹库和函数类别数
   - `ReIFunc` 在这两个类别中远大于另外两个方法
   - 指纹库越大，能学习的特征越多
   - 函数类别越多，能识别的内联种类越多 


# 6. LIMITATIONS AND FUTURE WORK
1. `ReIFunc` 仅在重复内联函数上的表现良好，不适应普通内联函数
2. 内联后，边界基本块中包含内联函数和调用函数，目前只支持粗粒度边界计算
3. 编译优化会导致 CFG 发生严重改变，进而影响子图同构算法难以识别


# Idea
1. 加快指纹数据库的检索
2. 对于召回率过低的改进
3. 