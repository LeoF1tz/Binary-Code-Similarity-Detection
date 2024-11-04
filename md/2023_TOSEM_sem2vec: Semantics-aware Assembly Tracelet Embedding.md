| **Title** | Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Huaijin Wang |
| **Institution** | Hong Kong University of Sience and Technology |
| **Venue** | TOSEM |
| **Year** | 2023 |


# ABSTRACT
1. 现有挑战
   - 现有模型着重于提取代码的语法结构、数据流和控制流，但这些信息不足以反应程序功能，并且面对编译优化性能会显著下降
2. 本文方法
   1. 将 CFG 拆分为 tracelet，并利用符号执行方法提取符号约束和相关信息
   2. 使用掩码语言模型计算符号的嵌入向量
   3. 使用图神经网络将 tracelet 嵌入聚合成函数的 CFG 级别嵌入


# CONCLUSION
1. sem2vec 基于 tracelet 的嵌入框架捕捉二进制代码的语义信息，从而生成嵌入向量
2. sem2vec 通过符号执行，掩码模型和图神经网络实现相似性检测
   1. 符号执行用于提取精确的语义约束
   2. MLM 和 GNN 用于聚合所提取的信息


# 1. INTRODUCTION
1. sem2vec 的核心思想
   1. 采用**符号执行和深度学习**混合，两者优势互补
   2. 符号执行
      - 优点：提取**精确语义级别**
      - 缺点：难以处理数据量大的情况，可拓展性地
   3. 深度学习
      - 优点：擅长学习代码表示的可扩展视图
      - 缺点：无法精确掌握语义
2. sem2vec 的步骤
   1. 将 CFG 分解为执行路径，即 Tracelet
   2. 对 Tracelet 执行 SE 以计算精确的语义特征
   3. 使用MLM 将每个符号特征转换为嵌入向量
   4. 使用GNN 将 Tracelet 嵌入向量聚合为整个 CFG 的嵌入向量
3. 实验结果
   - sem2vec 在多重比较设置中取得了大幅度领先，尤其是在跨编译器和混合设置下的鲁棒性超越了其他 SOTA
4. 贡献总结
   1. 一种从程序语义中学习二进制代码嵌入的新方法，即 `Tracelet`
   2. 一种通过结合可拓展的 tracelet 级别 SE、MLM 和 GNN 的实用工具，可以实现跨编译设置下的二进制代码相似性检测工具，即 `sem2vec`


# 2. PRELIMINARIES
![alt text](<images/2023_TOSEM_sem2vec: Semantics-aware Assembly Tracelet Embedding/img.png>)
1. 二进制代码嵌入步骤
   1. 可执行文件反汇编为汇编指令
   2. 构建 CFG
   3. 基本块级别嵌入
   4. 图级别嵌入，即用一个向量表示每个汇编函数
2. 基本块级别嵌入
   1. 核心思想
     - 将指令视为自然语言中的**句子**，即将操作码和操作数都视为单词
   2. 训练步骤
     - 模型 $M_b$ 从 指令 $i$ 中提取标记，并转换为嵌入向量 $v$
   3. sem2vec 的 tracelet 级嵌入基于预训练的 RoBERTa，即不再从头训练 $M_b$
3. 图级别嵌入
   - sem2vec 使用 GGNN 以聚合每个 tracelet 的嵌入


# 3. RESEARVH MOTIVATION
![alt text](<images/2023_TOSEM_sem2vec: Semantics-aware Assembly Tracelet Embedding/img-1.png>)
1. 不同的编译设置的影响，**功能一致，但是 CFG 可能发生显著变化**
   1. (a) 没有优化的 CFG，即 `O0`
   2. (b) 全面优化的 CFG，即 `O3`，显著复杂化
   3. (c) **虚假控制流**混淆，通过插入**恒真条件**，代码看似增加了复杂性，但**实际控制流未改变**
   4. (d) **扁平化**，使用 `switch-case`，功能相同，但是 CFG 复杂性增加
2. 符号约束的优势
   - 表示程序的输入输出关系和路径约束，而不是代码的具体结构形式，鲁棒性得到提高
3. 符号执行与DNN的结合
   - 本研究创新性地将符号执行生成的**符号约束**作为二进制代码嵌入模型的输入，旨在提取出更加精确且**稳健**的代码语义表示。通过这种方式，sem2vec能够在**不同的编译器优化、架构和混淆情况**下，**生成一致**且具有较高相似度的嵌入


# 4. DESIGN (todo)
![alt text](<images/2023_TOSEM_sem2vec: Semantics-aware Assembly Tracelet Embedding/img-2.png>)
1. sem2vec 整体框架
   1. 对 `exe` 文件进行反汇编，得到 `CFG`
   2. 对 `CFG` 使用


## 4.1. Tracelet-based USE
### 4.1.1. `Symbolic_Execution`
1. 调用栈维护
   - `Symbolic_Execution` 维护一个调用栈，当遇到 `ret` 指令时，算法会弹出最近的调用函数 `f`，并从调用点的下一条指令继续符号执行
2. 符号执行过程
   - `Symbolic_Execution` 在指定块 `B` 上执行符号执行
   - 它逐条解释机器指令，并相应的更新符号状态，即 `S` 到 `S'`
     - `S` 和 `S` 表示**程序当前执行状态**的符号化模型
   - 如果**首次加载**存储单元中的数据，为这些值创建**新的自由符号**
3. 路径约束
   - 表示在 `tracelet` 中从入口块 $B_0$ 到当前块 $B$ 必须满足的执行条件
4. 处理函数调用
   - 遇到函数点时，`Symbolic_Execution` 会**递归内联**这些被调用函数



### 4.1.2. `Traverse_Tracelet`
![alt text](<images/2023_TOSEM_sem2vec: Semantics-aware Assembly Tracelet Embedding/img-4.png>)
1. 执行过程
   - 从起始块 $B_0$ 开始，使用 **BFS** 遍历 CFG
2. 状态分叉
   1. 符号状态更新：sem2vec 会创建**两个新的符号状态**，分别表示条件的**真分支和假分支**；否则，只会生成一个符号状态，即 $S'_{next}$
      - $S'$ 会被分裂成两个部分 $S_j'$ 和 $S_i'$，并分别对应于两个后继块 $S_i$ 和 $S_j$
   2. 路径约束更新
      - $B_i$ 对应 $C_c \wedge C$
      - $B_j$ 对应 $\neg C_c \wedge C$
3. 最大符号状态：使用 `MAX_STATE` 限制符号状态的数量，**防止路径爆炸**
   - BFS 遍历每次最多允许生成 `MAX_STATE` 个 Tracelet
4. Tracelet 元数据：在遍历 `Tracelet` 时，同时收集元数据
5. 遍历的终止条件
   1. 函数的 `ret`，即当前函数结束：无法定位后续块，返回 `false`，**即结束当前路径的遍历**
   2. ~~无法解析的代码指针~~：无法定位后续块，返回 `false`，**即结束当前路径的遍历**
   3. 符号状态达到上限 `MAX_STATE`：检查栈是否为空
      1. 空调用栈：算法继续在当前函数内遍历
      2. 非空调用栈：表示路径**涉及跨函数调用**，需要回溯到函数 $F$ 的最近调用点
6. 使用 BFS 的原因
   - DFS 可能会陷入冗长路径中，使得获取的信息仅包含少量路径
7. 不可达路径修剪
   1. 在符号执行过程中，sem2vec 遇到条件路径时，会将符号状态分叉为两条路径（真分支和假分支），这在**默认情况下被视为两条都可达的路径**
   2. 使用**约束求解器**对不可达路径进行修剪


### 4.1.3. `Traverse_CFG`
![alt text](<images/2023_TOSEM_sem2vec: Semantics-aware Assembly Tracelet Embedding/img-3.png>)
1. 从目标函数 `F` 的入口块 $B_{entry}$ 开始，对 CFG 进行遍历
2. 使用堆栈存储待处理的基本块，对每一个块调用 `Traverse_Traclet`
3. 返回的 `tracelet` 集合加入集合 `R`，并将下一个块压入栈中


## 4.2. Tracelet Embedding
### 4.2.1. Preprocessing Symbolic Constraints
- i.e., `(eax < 0) AND (ebx > 5)`
   1. 符号约束的结构化表达
      - 左子树 `(eax < 0)`
      - 根节点 `AND`
      - 右子树 `(ebx > 0)`
   2. 中序遍历
      - `eax < 0`, `AND`, ` ebx > 0`
   3. 展平为序列：
      - `[eax, <, 0, AND, ebx, > 5]`
   4. 对数归一化
      - 将常量映射到一组**范围较小的不同值**


### 4.2.2. Pre-training Using Whole Word Masking (WWM)
1. 目的
   - 通过上下文信息学习符号约束的语义关系
2. 具体方法
   1. 对于符号约束 $C=(x_1, x_2, \cdots, x_n)$，随机选择 15% 的 $x_i$ 使用 $[MASK]$ 进行随机替换，得到 $C'=(x_1, [MASK], \codts, x_n)$
   2. 模型根据 $C'$ 预测 $C$
   3. 使用交叉熵损失计算误差
   4. 将 $C$ 转换为嵌入向量 $v$

### 4.2.3. Pre-training with Siamese Network
1. 核心思想
   - 使用孪生网络结构充分利用符号约束的语义信息，BERT 类模型往往只关注词汇级别，而忽略整体语义
2. 具体方法
   1. 构建约束对
      1. 等价约束对：同一源代码使用不同的编译设置生成**语法不同，但语义相同的**等价约束对
      2. 不等价约束度：随机选择和配对不同的约束生成**语法不同且语义不同**的不等价约束对
   2. 训练
      1. 输入约束对生成嵌入向量对
      2. 基于余弦计算相似度


### 4.2.4. Selecting Representative Constraints
1. 核心思想
   - 在 tracelet 的符号状态中选择具有代表性的约束，并将这些约束嵌入到一个最终的 tracelet 表示向量中
2. 具体方法
   - 1. 选择一个路径约束，和 K - 1 个寄存器输入输出约束
   - 2. 使用 HBMP 将 K 个向量压缩为 L 维的向量


## 4.3. CFG - level Embedding
1. 核心思想
   - 函数 F 的 CFG 由多个 tracelet 组成，这些 tracelet 形成了一个连通图 G，即每个 tracelet 对应 G 的一个节点
2. 具体方法
   1. 使用 GGNN 利用**节点间的相邻关系**来逐步更新嵌入向量，使每个节点的嵌入向量 $V_{tracelet}$ 包含**其邻居的信息**
   2. 使用 Set2set 将所有 $V_{tracelet}$ 聚合成一个 $V$，即 $V$ 表示函数整体的特征


# 7. DISCUSSION OF EFFECTIVENESS
## 7.1. Semantic Features
通过匹配相同源代码行的符号约束，`sem2vec` 提升了语义嵌入的质量


## 7.2. Structure-level Features
1. CFG 容易因编译设置的变化而发生显著改变
2. `sem2vec` 基于 CFG 构建新图 G，使得抗混淆能力增强


# Related Knowledge
1. 符号执行
   1. 定义
      - 一种程序分析技术，将程序中的**变量视为符号，而不是具体值**，用于探索程序的执行路径
   2. 目的
      - 提取程序的输入输出关系、约束条件和**路径约束**