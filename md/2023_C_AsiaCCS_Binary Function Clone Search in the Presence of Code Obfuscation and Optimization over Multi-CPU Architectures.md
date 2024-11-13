| **Title** | Binary Function Clone Search in the Presence of Code Obfuscation and Optimization over Multi-CPU Architectures |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Abdullah Qasem |
| **Institution** | Concordia University  |
| **Venue** | AsiaCCS |
| **Year** | 2023 |


# ABSTRACT
1. `BinFinder` 在 `tigress` 上实现了首次显著的二进制克隆搜索结果，即**具有抗混淆能力**
2. `BinFinder` 具有**混架构**下 BCSD 能力
~~3. `BinFinder` 基于神经网络~~


# CONCLUSION
1. `BinFinder` 能够在**混架构**下有效地处理**编译器优化**和**代码混淆**的影响
2. `BinFinder` 在多种场景下的性能显著优于现有方法


# 1. INTRODUCTION
1. 本文贡献点
   1. 


# 2. QUEST FOR RESILIENT FEATURES
~~文章没有明确相似函数间的差异值的计算方式~~
![alt text](<images/2023_C_AsiaCCS_Binary Function Clone Search in the Presence of Code Obfuscation and Optimization over Multi-CPU Architectures/img.png>)
1. 确定在**代码混淆和优化技术下**，（最）具有**鲁棒性**的特征
   1. `num_callers`：调用目标函数的函数数量
      - **目标函数被多少个其他函数所调用**，即反映了目标函数的**被依赖程度**（被动）
      - 75% 的相似函数具有相同的 `num_callers`
   2. `num_callees`：目标函数调用的函数数量
      - **目标函数内部调用了多少个其他函数**，即反映了目标函数的**依赖程度**（主动）
      - 目标函数可能存在**多次调用同一函数** >> 将 `num_unique_callees` 特征选入
      - `P(0)` 的值大于 0.6
   3. `num_lib_callees`：目标函数调用的 `lib` 函数数量
      - 91% 的相似函数具有相同的 `num_lib_callees`
   4. `num_unique_callees`：目标函数调用的唯一函数数量
      - `P(0)` 的值大于 0.7 
   - 存在某些**不相似的函数具有相同**的 `num_lib_callees` 或 `num_callers` 值，引入 `libcCalls` 和 `Constants` 进一步进行区分
     1. 两个函数的 `num_libc_callees` 值相同，但是具体调用的 `libc` 可能不同 >> `libcCalls`
     2. 两个函数所使用的 `Constants` 可能不同，作为**独特特征**的补充，并且其具有**相对较强**的抗混淆和抗混架构能力
2. 特征的鲁棒性的评估指标
   1. `P(0)`：一对相似二进制函数具有相同目标特征值的概率
   2. `diff_mean`：表明选定的特征值受到目标编译器优化或代码混淆的影响程度
   - 具有高鲁棒性的特征值应**同时**具有**高 `p(0)` 和 低 `diff_mean`**
3. 特征的鲁棒性的计算方式
   - 输入成对的相似函数的绝对差异值进入经验分布函数，计算 `P(0)` 和 `diff_mean`
     - i.e., $F_i = (O0, num_lib_callees, gcc, x86)$ 和 $F_j = (sub, num_lib_callees, clang, arm)$
     - 即在同一特征下，相似函数的不同情况
4. **使用 `VXE-IR` 应对混结构的不同汇编指令格式**


# 3. BINFINDER APPROACH
![alt text](<images/2023_C_AsiaCCS_Binary Function Clone Search in the Presence of Code Obfuscation and Optimization over Multi-CPU Architectures/img-1.png>)
- `BINFINDER` 的步骤
  1. 数据收集和生成
  2. 特征选择和表示
  3. 模型学习：
  4. 查询和结果


## 3.1. Preprocessing Selected Features
1. 标准化处理 `VXE-IR`
2. **统一**不同表示类型的特征，以保证具有**相同分布**
   1. 数值型  `Libccalls count`, `Unique Callees Count`, `Callees Count`, `Callers Count`
      - i.e., `count = 5` >> `List = {1, 2, 3, 4, 5}` 
   2. 列表型  `VEX-IR instructions`, `LibcCalls`, `Constants`
3. 调整权重
   - `num_callers`, `num_unique_callees` 和 `num_libc_callees` 在指令间的差距绝对差距不大 >> 对最后的相似性判断影响小 >> **扩大权重** >> **放大相对差距**
   - 通过**增加其特征向量维度**的方式扩大 `num_callers`, `num_unique_callees` 和 `num_libc_callees` 的权重，


## 3.2. Feature Representation
1. 对每一个特征使用**独立的向量转换器** >> 转换成 `one-hot` 向量
2. **串联**每个 `Tokenizer` 输出的向量 >> 整个二进制函数的向量表示


## 3.3. Siamese Neural Network Architecture
- 孪生网络结构由两个**共享参数且相同**的三层 MLP 组成


# 4. EVALUATION
## 4.2. Code Obfuscation
### 4.2.1. O-LLVM
![alt text](<images/2023_C_AsiaCCS_Binary Function Clone Search in the Presence of Code Obfuscation and Optimization over Multi-CPU Architectures/img-2.png>)
1. 训练两个模型测试
   1. `M1` 仅在优化样本上训练，**从未接触过混淆样本** >> 表现更加稳定
   2. `M2` 使用混淆和优化的混合数据集进行训练 >> **可能学习到由混淆技术引入的噪声**
2. 结论
   - `BinFinder` 在特定数据集上会因为其对 `BinFinder` 选用的特征修改多少而产生性能差异
   - `BinFinder` 可以在未见过任何混淆样本的情况下，以高精度识别由 `O-LLVM` 引入的混淆二进制函


### 4.2.2. tigress Obfuscator
- 使用 `M1` 进行测试
1. `O0`
   - 在 `Flatten` 和 `virtualization` 下表现较差，但是其他三种情况下，表现良好，有一定抗混淆能力
2. `O3` 
   - 编译器优化等级的提升，会增强混淆效果，进而导致模型能力降低


# 5. SEARCHING AGAINST ALL BINARIES
- 本文所选用的特征 `num_constants`, `libcCalls`, `Callers` 和 `Callees` 会受到混架构的影响

# Related Knowledge
##  1. Empirical Distribution Function
1. Definition
   1. 样本数据 = $\{x_1, x_2, \cdots, x_n\}$
   2. $F_n(x) = \frac{1}{n} \sum^n_{i=1} I(X_i \leq x)$
   3. 表示样本中小于等于 $x$ 值的比例 
2. Purpose
   - 经验分布函数在每个数据点处累加频率，逐步构建出样本的分布情况