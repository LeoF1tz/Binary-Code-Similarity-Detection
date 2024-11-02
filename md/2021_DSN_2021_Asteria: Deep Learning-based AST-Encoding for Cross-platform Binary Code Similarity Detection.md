| **Title** |  Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection |
|----------|----------------------------|
| **Author** | Shouguo Yang |
| **Institution** | Chinese Academy of Sciences |
| **Venue** | DSN |
| **Year** | 2021 |


# ABSTRACT
1. 现有问题：捕捉语义相似性部分存在不足
1. 本文方法：利用 AST 获取函数语义信息，并使用 Tree-LSTM 进行编码
2. 模型能力
   1. 二进制函数相似性检测
   2. 漏洞搜索


# CONCLUSION
1. 本文方法：将 **AST** 作为函数特征，使用 **Tree-LSTM** 进行编码，以及采用 **Siamese** 网络进行相似性计算


# 1. INTRODUCTION
1. 现有挑战：**架构复杂性**导致指令多样化
2. AST 的优势：**语义信息**丰富
3. 本文方法：使用 Tree-LSTM 网络和 Siamese 网络进行 AST 编码与相似性计算
4. 主要贡献
   1. ASTERIA，即通过 Tree-LSTM 网络在二进制级别编码 AST 为语义表示向量
   2. 实现并开源了 ASTERIA 模型，可以在 BCSD 任务中执行跨架构搜索
   3. ASTERIA 可用于实现漏洞搜索


# 2. PRELIMINARY
## 2.1. Abstract Syntax Tree
![alt text](<images/2021_DSN_2021_Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection/img.png>)
1. 叶节点：变量和常量
2. 非叶节点
   1. 语句节点：控制流
   2. 表达式节点：计算流


## 2.2. AST vs. CFG
1. AST 具有**架构稳定性**
   1. AST 由中间表示生成
   2. CFG 在不同架构下的差异很大


## 2.3. Tree - LSTM
1. Tree - LSTM 可以用于处理树形结构，即满足 AST 的结构
2. Tree - LSTM 在语义分析任务中，性能优于 LSTM
3. 使用 Binary Tree - LSTM
4. Tree - LSTM 的计算流程：自下而上，即从叶节点开始计算，根据子节点的信息得到当前节点


## 2.4. Problem Definition
![alt text](<images/2021_DSN_2021_Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection/img-1.png>)
1. 二进制函数对 $<F_1, F_2>$ 转换成AST对 $<T_1, T_2>$
2. AST 的定义：$T = (V,E)$
   1. $V$ 表示顶点，并且会根据表将 $V_i$ 转换为对应的 $LABEL$
   2. $E$ 表示边，$l_{kj}\in E$ 表示顶点 $k$ 是顶点 $j$ 的父节点
3. 相似性定义
   1. 模型对两个二进制函数 $<F_1, F_2>$ 的相似性计算基于其对应的 AST 对 $<T_1, T_2>$
   2. 模型的输入为 AST 对 $<T_1, T_2>$
   3. 模型的输出为相似性得分，1 表示完全相似，即两者为同源啊函数；0表示完全不同，即两者为不同源函数


# 3. METHODOLOGY
## 3.1. Overview
![alt text](<images/2021_DSN_2021_Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection/img-2.png>)
1. 预处理
   1. 输入一个二进制函数对 $<F_1, F_2>$
   2. 反汇编和反编译
   3. 提取 AST $<T_1, T_2>$
2. AST 数字化和格式转换
   1. 将每个 AST 映射为一个**整数值**
   2. 转换格式为**左子右兄弟**
3. 编码和相似性计算
   1. 使用 Tree-LSTM 转换成向量
   2. 使用 Siamese 网络计算两个向量之间的相似性
4. 相似性校准
   1. 使用**函数调用关系**对 AST 相似性进行校准，得到最后的函数相似性


## 3.2. AST Similarity Calculaton
### AST Similarity Calculation
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


### Siamese Network
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


## 3.3. AST Similarity Calibration
![alt text](<images/2021_DSN_2021_Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection/img-3.png>)
1. 调用函数特征
   - 调用函数的数量具有架构无关性，即在不同架构中保持稳定
2. 为什么可以用于相似性校准
   - 同源函数由于**共享相同的源代码**，通常具有**相同的调用函数数量**
   - 但是，可能会发生**函数内联**，导致同源函数的调用函数数量不一致 
3. 对于内联函数的解决方法
   - 对函数的**指令数进行筛选**，设置一个阈值，低于该阈值则将这个函数视为内联函数，并不记录调用函数数量
4. 具体方法
   1. 过滤指令条数低于阈值的函数
   2. 计算两个函数的调用函数相似度，作为校准相似度
      - $S(C_1,C_2)=e^{-|C_1-C_2}|$
      - $C_1$ 和 $C_2$ 表示 $F_1$ 和 $F_2$ 过滤后的调用函数数量
      - 两个函数的**调用函数数量越接近，这个分数越高**
   3. 计算最后的函数相似性得分
      - $F(F_1, F_2)=M(T_1,T_2)*S(C_1, C_2)$


# 4. EVALUATION
## 4.1. ROC Comparison
1. 实验设置
   1. 混合架构实验：随机从**任意架构**中选择函数对
   2. 成对架构实验：测试**特定架构**组合的性能，例如 `ARM-PPC`
![alt text](<images/2021_DSN_2021_Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection/img-4.png>)
2. 混合架构实验：低假阳性率下表现出高真阳性率
![alt text](<images/2021_DSN_2021_Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection/img-5.png>)
3. 成对架构实验：AUC 值遥遥领先
4. 原因分析
   1. 与 `Gemini` 相比，体现了 `AST` 带来的性能提升
   2. 与 `Diaphora` 相比，体现了 `NLP` 技术，即 `Tree-LSTM` 和 `Siamese` 带来的提升


## 4.2. Impact of Model Settings
![alt text](<images/2021_DSN_2021_Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection/img-6.png>)
1. 嵌入大小：AST 被转换成的嵌入向量的维度
   - 嵌入大小为16时，模型性能最佳，最高AUC值0.985
   - 嵌入向量过小，不足以完整表达节点的语义信息；过大，则可能发生过拟合
![alt text](<images/2021_DSN_2021_Asteria: Deep Learning-based AST-Encoding for Cross-platform Binary Code Similarity Detection/img-7.png>)
2. Siamese：比较余弦距离和二元分类的相似性计算方法
   - 余弦距离（Regression）
   - 二元分类（Classification）
   - Classification 的结果（AUC 值）显著高于 Regression
3. 叶节点计算：叶节点没有子节点，需要对它们进行初始化
   - 全 0 初始化
   - 全 1 初始化
   - 使用全 0 初始化的 AUC 值更高


## 4.3. Computational Overhead
1. AST 大小
   - 绝大部分函数（97.4%$）对应的 AST 小于 200
2. 时间消耗：只考虑 AST 大小小于 300 的函数
   - 离线阶段：AST 的提取、预处理和编码
     - 反编译的耗时，ASTERIA 显著高于其他方法；其余两个阶段类似
   - 在线阶段：相似性计算阶段
     - ASTERIA 的计算时间极短


# 5. VULNERABILITY SEARCH
1. 编码
   - 使用 TRRE-LSTM 对固件数据集和漏洞数据集分别进行编码，得到其对应的向量表示
2. 相似性计算
   - 使用 SIAMESE 网络计算固件数据集中的函数 $f$ 和漏洞数据集中的漏洞 $v$ 的相似性得分，即 $r_{f,v}$
3. 筛选
   - 使用 `Youden` 选择最佳的相似性分数阈值 `0.84`，过滤出候选漏洞函数，高于这个，$r_{f,v}$ 高于这个阈值，则认为 $f$ 可能是 $v$ 的同源漏洞函数
4. 手动分析
   - 对于 3 得出的函数再加以手动分析得到最后的漏洞
5. 结果
   - ASTERIA 检测出 75个漏洞函数，说明其具有漏洞检测的能力