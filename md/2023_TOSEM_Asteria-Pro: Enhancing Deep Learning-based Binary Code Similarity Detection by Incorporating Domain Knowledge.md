| **Title** | Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge |
|----------|-------------------------------------------------------------------------------------|
| **Author** | SHOUGUO YANG |
| **Institution** | University of Chinese Academy of Sciences |
| **Venue** | TOSEM |
| **Year** | 2023 |

# ABSTRACT
1. 现有问题
   - Aster 中存在**时间开销高和精度不足**的问题
2. 创新架构
   - 预筛选：通过去除不相似函数，以减少计算量
   - 重新排序：提升后选函数的排名，以增加模型精度
3. 模型效果
   - 预筛选模块将计算时间减少了 96.9%
   - 重新排序模块使得 MRR 和 Recall 分别提升了 23.71% 和 36.4%


# CONCLUSION
1. 创新点
   1. 引入领域知识，即函数调用关系
   2. 预筛选：在函数编码之前广泛利用函数名称信息，加速编码过程
   3. 重新排序：在函数编码后利用函数调用结构对编码相似性得分进行校准，确保同源函数的排名得以提升


# INTRODUCTION
1. ASTERIA
   - 1. 模型结构
     - 使用 `Tree-LSTM` 结合 `Siamese` 结构对 `AST` 进行编码，以捕捉函数的语义特征
   - 2. 训练方式
     - 输入同源和非同源函数对
     - 使用 CFG 校准 AST 相似度
   - 3. 缺陷
     - 1. 时间成本高：对于 AST 编码需要大量时间
     - 2. 误报率过高：对于具有相似 AST，但非同源函数时，ASTERIA 的性能并不如意
2. ASTERIA - PRO 的组成构建
   - **基于领域知识的预筛选模块**
   - 基于深度学习的相似性检测模块
   - **基于领域知识的重新排序模块**


# 2. BACKGROUND
## 2.1. Abstract Syntax Tree
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img.png>)
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-1.png>)
1. AST 的结构
   1. 叶节点：变量或者常量
   2. 节点：被分为 **Statement** 和 **Expression** 两类
      1. Statement: 控制函数的执行流程
      2. Expression: 执行计算操作
2. AST 的优势：具有**架构稳定性**，即在不同架构下差距不大


## 2.2. Tree-LSTM
1. Tree-LSTM 是 LSTM 针对树结构进行的优化，且更适合用于处理复杂的语义关系
2. 在 Tree-LSTM 的两种结构中，使用 Binary Tree-LSTM：更能保留子节点的顺序信息


## 2.3. Function Call Compilation Optimization
1. 函数内联：将**被调用函数的代码直接插入到调用函数**中的优化方法，从而避免实际的函数调用
2. 内建函数：由编译器直接实现的特殊函数，映射为目标架构中的低级指令，能够直接访问硬件资源


# 3. PRELIMINARY STUDY
## 3.1. Evaluation Benchmark
### Dataset
1. 架构选择：x86, x64, arm, ppc（跨架构
2. 删除用于测试软件功能的函数
3. 同源函数和非同源函数的定义
   - 具有相同函数名视为同源函数（包含不同架构
   - 具有不同函数名视为非同源函数
4. 函数对构建
      - 对于每个函数 $F^B_N$ 包含 $M$ 个非同源函数和 3 个同源函数


### Metrics
1. TPR: 保留同源函数的能力
2. FPR: 过滤非同源函数的能力
3. 目标：在保持低 FPR 的同时 TPR 尽可能高


## 3.2. Candidate Features Evaluation
### Feature Selection
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-2.png>)
1. 对 CFG 特征和 AST 特征进行筛选，筛选最具有分辨价值的特征用于预筛选
2. 筛选条件：是否能在高 TPR 下保持较低的 FPR，即 **$F_{score}$**
   - $F_{score}=\frac{1}{\frac{1}{TPR}+FPR}$
3. 评估结果
   - 选择**NCL、调用指令数量和字符串常量**作为预筛选指标

# 4. METHODOLOGY OVERVIEW
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-3.png>)
1. DK-based Prefiltration
   - 通过领域知识中的语法特征来快速排除不相关的函 
2. DL-based Similarity Calculation
   - Tree-LSTM 模型将函数的 AST 转化为向量表示，再利用孪生网络进行相似性评分
3. DK-based Re-ranking
   - 利用函数的调用关系等轻量结构特征，重新排序模块对候选的同源函数进行再排序


# 5. DK-BASED PREFILTRATION
## 5.1. Exploitation Challenges
1. 函数名称可能与源代码中的原始名称不同
2. 在调用图中，某些函数（通常称为叶节点）没有调用其他函数，因缺乏显著特征，可能在筛选中被剔除
3. 由于函数内联、固有函数替换等技术手段，二进制目标函数中的函数调用可能与源代码不一致


## 5.2. Definition of NCL
1. NCL 基于 CG 和 DST 构建
   - CG 即函数调用图图，点为函数，边为调用关系
   - DST 即动态符号表
     - i.e., 如果目标函数调用了一个外部函数 `strcpy`，那么 `strcpy` 会被保存在 DST 中 
2. NCL 的定义
   1. Named Callee List，即命名被调用列表，用于表示某个目标函数直接调用的所有函数集合
   2. 出目标函数调用的所有**保留名称**的函数
   3. $NCL_f=\{v|v\in V,v\in DST,(f,v)\in E\}$
      - $v\in V$ 表示被调用函数属于函数调用图中的某一个函数
      - $v\in DST$ 被调用函数存在于 DST 中
      - $(f,v)\in E$ 目标函数和被调用函数在 CG 中存在边，即两者具有调用关系
3. 应对挑战 1，即函数名发生变化
   1. 使用工具 `cxxfilt` 恢复
   2. 定义某些规则处理其余函数


## 5.3. Filtration Algorithm
1. `UpRelation` 算法应对挑战 2 和挑战 3
   - 核心思想：利用**调用图的上下文信息**，特别是叶节点的父节点来匹配相似的叶节点，以此筛选出潜在同源的候选函数
   - 目标：在尽可能保留同源函数的情况下，**过滤非同源函数**
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-12.png>)
2. 输入参数
   1. **Vulnerable Function** $f_v$，脆弱函数，即希望从候选函数列表中找到与该函数相似的脆弱函数（源函数）
   2. **Target Function List** $TFL$，候选函数列表，用于存储**需要与目标函数进行相似性匹配的函数**
   3. **Threshodls** $T_{NCL}, T_{callee}, T_{string}$，三个阈值，用于判断 NCL 相似性，被调用函数相似性和字符串相似性（三个不同层次）
3. 输出参数
   - **Vulnerable Candidate Function List** $VFL$，最后筛选出的脆弱候选函数列表，仅保留可能与目标函数相似的函数 
4. 算法步骤
   1. 初始化候选表
      - $TFL$ 赋值给 $VFL$
   2. NCL 相似性过滤
      1. 检查目标函数 $f_v$ 的 $NCL$ 是否非空
      2. 遍历每个候选函数 $f$，计算其 $NCL$ 与 $NCL_{fv}$ 的相似性 $s$，如果低于阈值 $T_{NCL}$，则从 $VFL$ 中删除 $f$
   3. 被调用函数数量过滤
      1. 检查目标函数 $f_v$ 的 $Callee_{fv}$ 是否 $>0$
      2. 遍历每个候选函数 $f$，计算其 $Callee_f$ 与 $Callee_{fv}$ 的相似性 $s$，如果低于阈值 $T_{callee}$，则从 $VFL$ 中删除 $f$
   4. 叶节点匹配，不满足 2 和 3 即该节点为**叶节点**；叶节点无法通过自身的特征，即调用函数进行判断，则使用**被调用**特征进行判断
      - 步骤 2 和 3 是使用*调用函数的信息*进行判断，即基于 $fv$ 自身的特征筛选
      - 步骤 4 时，由于是叶节点，自身并不调用其他函数，即自身信息缺失或不足，使用**调用自己的函数作为信息**进行判断，即基于调用 $fv$的函数的相似性进行筛选
      1. 创建空集 $FL'$ 存储过滤后的候选函数
      2. 遍历目标函数 $fv$ 的所有调用函数 $caller$，并递归调用 $UpRelation$ 算法来匹配相似的调用函数 $caller'$
      3. 对于每一个匹配的 $caller'$，将其所有被调用函数加入到 $FL'$
      4. 将 $FL'$ 赋值给 $VFL$
   5. 字符串相似性判断
      1. 遍历 $VFL$ 中的每个候选函数 $f$，计算其字符串集合 $StrCons_f$ 与目标函数 $StrCons_{fv}$ 的字符串相似性比率 $s$，小于阈值 $T_{string}$，则删除该候选函数
   6. 返回 $VFL$，其仅包含所有通过过滤条件的函数


# 6. BL-BASED SIMILARITY CALCULATION
## 6.1. Tree-LSTM Encoding
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-4.png>)
1. 计算过程（**逐层递进**的计算关系）
   1. 缓存状态 $u_k$
      -  $u_k = tanh(W^u e_k+(U^u_l h_{kl}+U^u_r h_{kr})+b^u)$
      - 结合当前节点嵌入 $e_k$ 和子节点隐藏状态，再使用 $tanh$ 激活
   2. 细胞状态 $c_k$
      - $c_k=i_k\odot u_k+(c_{kl}\odot f_{kl} +c_{kr}\odot f_{fr})$
      - 使结合缓存状态和子节点的细胞状态，并使用遗忘门过滤信息，逐步融合树结构的信息 
   3. 隐藏状态 $h_k$
      - $h_k=o_k\odot tanh(c_k)$
      - 结合细胞状态和输出门，形成当前节点的**最终表示**，再往上传递，直到根节点
2. 计算方式
   1. 从叶节点开始**自下而上**进行编码
   2. **根节点的隐藏状态** 即 $h_{root}$ 作为其所属 AST 的整体向量表示，用于后面的相似性计算

## 6.2. Siamese Calculation
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-5.png>)
1. 作用
   - 比较两个函数 AST 的相似性，即输出相似性得分
2. 特点
   - 由两个**共享参数的 Tree-LSTM 模型**组成
3. 计算过程
   1. 向量化：Tree-LSTM 分别对两个 AST 进行编码，得到它们的向量表示
   2. 计算操作：对两个编码向量执行减法（绝对值）和 Hadamard 乘积操作
   3. 结果处理：结果向量连接成一个更大的向量，并通过 softmax 层归一化成二维向量，代表**相似度和不相似度**
      - 即**输出向量的第二个值作为 AST 的相似度**，并后续用于重新排序
4. 训练过程
   - 输入格式为 $<T1, T2, LABEL>$
   - $T1$ 和 $T2$ 为函数对
   - $LABEL$ 表示它们是否为同源函数
5. 推理过程
   - 输出格式为 $<不相似分数, 相似分数>$


# 7. DK-BASED RE-RANKING
1. 核心任务
   - 对 Tree-LSTM 网络输出的前 $K$ 个候选函数进行重新排序
2. 方法
   - 引入领域知识，即函数调用关系，它**提供上下文关系**


## 7.1. Motivated Example
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-6.png>)
1. 算法基础
   - 如果一个函数 $F$ 调用了函数 $CF$，那 $F$ 的同源函数 $F'$，也应该调用 $CF$ 的同源函数 $CF'$ 
2. 对于 $F_1$ 的两种调用函数检测
   1. 已知名称的函数 $C^{F_1}_n$ 使用**函数名称**匹配
   2. 匿名函数 $C^{F_1}_a$ 基于**深度学习**方式进行检测 


## 7.2. Relational Structure Match Algorithm
1. 基于源函数 $F$ 是否有被调用函数，进行两种操作
   1. 有调用函数
       1. 算法提取所有被调用函数，并基于此构建**混合被调用数据集（MCFS）**
         - 混合被调用数据集包含两种类型，即**命名被调用函数和匿名被调用函数**
       2. 使用 MCFS 算法计算目标函数与候选函数之间的相似性，得到**新的匹配分数**，并基于此进行**重新排序**
   2. 无调用函数
      1. 删除所有**具有调用函数的候选函数**
      2. 重新排序


### MCFS
- MCFS中包含两种类型的函数，即**命名被调用函数和匿名被调用函数**


### Match Score Calculation
1. 命名被调用函数匹配
   1. 根据**函数名称**将其与每个候选函数的命名被调用函数进行匹配
   2. 计算候选函数 $F_i$ 对应的**匹配数量** >> $N^{F_i}_n$
2. 匿名被调用函数匹配
   1. 利用**基于深度学习的相似性检测**来计算目标函数的匿名被调用函数与所有候选函数的匿名被调用函数之间的相似性分数
   2. 选择候选函数 $F_i$ 所有匿名被调用函数中的最大值 >> $S^{F_i}_{aj}$ >> 累加每一个源函数的匿名函数对应的最相似匿名函数 >> $\sum S^{F_i}_{aj}$
3. 候选函数 $F_i$ 的最终匹配分数
   - $M_{F_i}=N^{F_i}_n+\sum S^{F_i}_{aj}$


### Match Score-based Re-ranking
- $F_i$ 最终的重排名得分
  - $S^{re-rank}_{F_i} =\alpha * N_{F_i} + \beta * M_{F_i} $
- 根据 $S^{re-rank}_{F_i}$ 对所有候选函数 $F_i$ 进行**降序排列**


# 8. EVALUATION
## 8.5 Comparison of Similarity Detection Accuracy (RQ1)
### Cross-architecture Evaluation
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-7.png>)
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-8.png>)
1. 在任务-C 中，Asteria-Pro 在所有架构组合中与 Asteria 表现相当，但其 AUC 总体高于其他基线方法
2. 在任务-V 中，Asteria-Pro 的 MRR 和 Recall 指标显著高于 Asteria 和其他基线方法
3. ASTERIA 误报原因分析
   1. 代理函数的**相似语法结构**，难以区分
   2. 不同编译器可能用不同的内建指令替换标准库函数，使得 ASTERIA - PRO 的预筛选和重排序模块失效


### Cross-compiler Evaluation
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-9.png>)
- Asteria-Pro 在跨编译器测试中显著优于基线方法


## 8.6 Performance Comparison (RQ2)
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-10.png>)
1. 特征提取时间
   - ASTERIA 需要基于 AST，而 AST 的提取需要进行二进制反汇编和**反编译**，因此较慢 
2. 相似性计算时间
   - 这里 ASTERIA-PRO 由于引入了筛选模块，故计算时间大幅减少


## 8.7. Ablation Experiments (RQ3)
1. 研究目的
   - 评估 Asteria-Pro 的预筛选和重新排序的**独立贡献**
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-11.png>)
2. 预过滤
   - 性能影响：MRR、Recall@Top-1 和 Recall@Top-10 分币提升了 12.26%, 16.29% 和 5.93$，**性能大幅提升**
   - 时间开销：相比不使用预过滤的 ASTERIA，**搜索时间缩短 96.94% **
3. 重新排序
   - 性能影响：MRR、Recall@Top-1 和 Recall@Top-10 分别提升了 20.16%、31.51% 和 3.76%，**性能大幅提升**
   - 时间开销：平均增加 0.13 秒的额外时间，开销较小
4. 使用基线方法融合预筛选和重排序
   - **显著提高**了这些方法的准确性


## 8.8. Configurable Parameter Sensitivity Analysis (RQ4)
1. 可调参数
   1. 过滤模块的阈值 $T_{NCL}$
   2. 重排序模块的权重参数
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-13.png>)
2. 过滤阈值
   1. 阈值越高，筛选越严格，可以过滤更多的非同源函数，但是 Recall 也随之下降
   2. 0.1 是最佳阈值，它在保持了较高的 RECALL 值的同时，过滤了足够多的非同源函数
![alt text](<images/2023_TOSEM_Asteria-Pro: Enhancing Deep Learning-based Binary Code Similarity Detection by Incorporating Domain Knowledge/img-14.png>)
3. 权重参数
   1. 随着 $\alpha$ 的增加，模型性能会随之下降
   2. $\alpha = 0.1$ 为最佳阈值
   3. $\alpha = 0$ 表示只是用预过滤和 Asteria，其与 $\alpha=0.1$ 的结果相比，说明了**重排序模块增强了模型的性能**


## 8.9. Real World Bug Search (RQ5)
Asteria-Pro 以 91.65% 的高精度检测出 1,482 个脆弱函数


# 10. DISCUSSION
## 10.1. How does the re-ranking module solve the function inline issues?
1. 将源函数的所有被调用函数和目标函数的所有被调用函数进行匹配
2. 若某些函数被内联，剩余函数仍然能够提供高相似分


## 10.2. What is the Design Difference between Pre-Filtering, Re-Ranking and SCA Tools
1. Asteria - Pro 使用的是**局部上下文关系**
   - Modex 使用的是整个图
2. Asteria - Pro 使用父节点的谱系关系帮助识别相似性
   - LibDB 依靠的是字符串文本和函数名称


## 10.3. What Will Asteria-Pro Perform on Cross-Optimization Settings?
Asteria - Pro 基于 AST，而 AST 在交叉优化下可能会发生很大变化，因此，Asteria - Pro 可能在交叉优化下的性能会显著下降