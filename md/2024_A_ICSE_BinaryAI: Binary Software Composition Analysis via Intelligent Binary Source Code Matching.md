| **Title** | BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Ling Jiang |
| **Institution** | University of Science and Technology |
| **Venue** | ICSE |
| **Year** | 2024 |


# ABSTRCAT
1. 现有方法的局限
   - 传统的 SCA 技术主要依赖于基本语法特征（如字符串文字），这些特征在大型TPL数据集中容易出现冗余，导致无法有效区分不同的TPL组件，导致误报增加和召回率下降
2. BinaryAI 的方法
   1. 第一阶段：使用 **TRANSFORMER** 生成函数级别的嵌入
   2. 利用**链接时局部性**，结合**函数调用关系**增强匹配的精度


# CONCLUSION
1. BinaryAI 采用了基于 `Transformer` 的模型，通过学习代码语言的基于标记的句法特征来生成函数嵌入，即通过分析代码的标记（token）级别的句法特征，以生成每个函数的嵌入表示
2. BinaryAI 利用基于局部性的匹配方法来丰富语义特征，即结合了函数调用的局部性和关联性
3. 在识别出匹配的源函数后，BinaryAI进一步检测目标二进制文件中的复用TPL组件
4. 实验结果和模型比较


# 1. INTRODUCTION
1. 具体方法
2. 贡献点
   1. 首次将函数级别的二进制源代码匹配应用于二进制到源代码的 SCA，并训练了一个基于 TRANSFORMER 的模型来检索相似的源函数
   2. 在 BinaryAI 中提出了两个阶段的二进制源函数匹配，并使用链接时局部性提高了前 k 个函数匹配精度
   3. BinaryAi 的性能比 SOTA 更高

# 2. BACKGROUND
## 2.1. Software Composition Analysis
1. 软件组成分析的概念
   -  从大型 TPL 数据集中提取软件特征来构建 SCA 数据库，然后利用代码克隆检测来识别 TPL 和二进制文件之间的相似特征。随后，如果相似特征的数量超过预定义的阈值，则将这些 TPL 识别为复用的组件
2. 二进制 SCA 的主要应用场景
3. 二进制 SCA 的类别及应用场景
   1. 二进制到源代码 SCA
   2. 二进制到二进制 SCA


## 2.2. Motivation
1. 二进制到二进制 SCA 的局限性：**TPL 数据集的规模**。具体而言，自动编译过程的复杂性使得**只有包管理器维护的少数 TPL 源代码包可以自动编译为多个版本**的二进制文件
2. 二进制到源代码 SCA 的限制
   1. 现有的二进制到源代码 SCA 工具主要依赖**基本句法特征，如字符串字面量**。这种方法存在显著的**冗余问题**，导致相同或相似的**特征出现在多个 TPL 中**，从而降低了 SCA 的识别准确性
      - `407 Proxy Authentication Require` 表示 `HTTP` 错误的字符串字面量
   2. 缺少显著的特征
      - 基本依赖于字符串常量和函数名称
   3. 现有方法提取字符串并不具有鲁棒性


# 3. APPROACH
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img.png>)
1. 过程
   - 特征提取 >> 基于嵌入的函数搜索 >> 基于局部性的匹配 >> 第三方库的检测 
2. 两阶段
   1. 训练一个基于 Transformer 的模型，以学习基于标记的句法特征，并从语料库中为每个查询的二进制函数检索出前 k 个最相似的源函数
   2. 利用额外的**结构化表示（例如链接时的局部性）**来捕捉语义特征，从前 k 个候选函数中匹配出准确的源函数；当匹配的源函数比例**超过预定义的阈值时**，BinaryAI 识别出目标二进制文件中复用的 TPL 组件


## 3.1. Feature Extraction
1. 源函数的特征提取
   1. 使用 `git` 作为版本控制工具。从每个项目的所有版本中收集 C/C++ 源代码文件，并使用文件的**哈希值来唯一标识每个文件**
   2. 采用 `tree-sitter` 工具来构建源代码的 AST，借助其 C/C++ 解析器提取唯一的源函数
   3. 使用倒排索引，记源函数的文件所在地址和源函数对应的 TPL
2. 二进制函数的特征提取
   1. 使用 `Ghidra` 进行反编译
      1. 提取**函数、数据结构和其他信息**
      2. 反编译为类 C 的**伪代码**
      3. 提取 **`bin_rva`，即相对虚拟地址**，以用于表示链接时的局部性
      4. 提取**函数调用图**，提供函数间的通信结构信息
   2. `BinaryAI` 假设输入的二进制文件已经将调试信息和符号信息剥离


## 3.2. Embedding-based Function Retrieval
1. 核心思想
   - 将二进制和源代码的嵌入表示映射到**同一个向量空间**中
2. 具体方法
   1. 使用带标签的数据（二进制-源代码函数对）和**大语言模型生成嵌入**
      - 稳重使用 `Pythia` 
   2. 利用**对比学习**方法，使得相似对在向量空间中更接近
      - **增加负样本的数量**使得模型可以学习到更有区分价值的特征
   3. 使用 `CLIP` 作为损失函数
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img-1.png>)
3. `CLIP` 实现**对比学习**的具体步骤
   1. 将**源代码和二进制代码 Tokenization**
   2. 使用 `Pythia` 模型转换成向量表示
   3. **正负样本对构造**
      1. $e^b_i$ 和 $e^N_i$ 被认为是正样本，即**蓝色**
      2. $e^b_i$ 和 $e^N_j$ 被认为是负样本，即**白色**（其中，$i\neq j$
      - 这里的负样本数量可以达到 $N\times (N-1)$
      - **值得参考**
   4. 损失函数 $L_{CLIP}$
      1. 计算 $B\to S$ 的损失 $L_{bin}$
      2. 计算 $S\to B$ 的损失 $L_{source}$
      3. 最终损失：$L_{CLIP} = \frac{{L_{bin}}+L_{source}}{2}$
      - 1 和 2 的区别是**嵌入顺序**，确保**两种匹配方**向都进行优化
   5. ~~应用 $MoCo$ 以构建动态字典，**增加负样本数量**~~
      - 文中只是提及，并没有解释如何应用
4. ~~这部分阐述了离线训练后，实现**在线匹配**的过程~~
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img-2.png>)
5. 使用**相对虚拟地址**作为二进制函数的唯一标识符
   - **值得参考**
6. 之前的**倒排索引**使得可以溯源每个匹配源函数的出处


## 3.3. Locality-driven Matching
1. 源文件到二进制文件编译过程的三个基本特性（`.c` >> `.o` >> `Binary File`）
   1. 源文件 `.c` 经过编译后形成单个目标文件 `.o`，其中**包含同一源文件中的所有源函数**的二进制版本
      - i.e.,
      - `file.c` 包含 `funA` 和 `funC`，其编译后的 `file.o` 文件中包含 `funA` 和 `funC`
   2. 目标文件被连续地链接到二进制文件中，目标文件代码部分中的所有函数（即机器代码格式的二进制函数）保留了**相对局部性**
      - i.e.,
      - `file.o` 文件中 `funA` 在 `funB` 之前，链接时，`funA` 的相对位置也会在 `funB` 之前，即**局部性**
   3. 由于 C/C++ 模板函数和条件编译，**一个源文件中的一个源函数可以对应多个二进制函数** （ `1-to-n`
      - i.e., 
      - **模版函数**会根据传入参数的不同，i.e., `int/float`，生成不同的版本
      - **条件编译** `if/ifef` 也会导致多种版本 **[1]**
2. 由 1 推断，**由同一源文件编译而成的二进制函数在二进制文件中的链接时局部性是连续的**
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img-3.png>)
3. `MatchFuncPairs` 实现过程
   1. 初始化
      1. `Input: bin2src_topk` 即 3.2 阶段后输出的二进制函数和源函数的前 k 个相似集合
      2. `Output: bin2src_match` 最终的二进制到源代码的匹配对
      3. `file2pairs` 存储**每个文件**中的二进制函数与相似源函数的对
      4. `intervals` 存储每个文件的**连续函数区**间
   2. 构建**文件-函数对**索引
      1. 遍历 `bin2src_topk` 中的每个二进制地址 `bin_rva` 和其对应的源函数列表 `similar_funcs`
      2. 对每个源函数 `src_func` 和其所属的源文件 `src_files`，将 `bin_rva` 和 `src_func` 的对应关系添加到 `file2pairs` zhong
      3. 初始化每个二进制函数对应的最相似源函数
   3. 生成并存储连续区间
      - 确定每个源文件 `file` 中含有的每个二进制源函数对 `bin2src_pairs` 的每个二进制函数对应的**最大连续区间**
   4. 排序区间
      -  按照排序优先级：`起始位置低 > 结束位置高 > 包含函数数量多` 进行排序
   5. 筛选排序后的区间
      1. 该区间的起始位置必须是前一个区间的结束位置之后，防止覆盖
      2. 使用函数调用图进行矫正（二进制函数和源函数的调用关系应该是一一对应的
         - `binA` 调用 `binB`，则其对应的 `srcA` 也应该调用 `srcB` ，反正亦然 
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img-4.png>)
4. `MaxFileInterval` 实现过程
   1. 作用：计算**最大连续区间**
   2. 按照**二进制函数的相对虚拟地址**递增顺序排序
   3. `interval` 记录当前的区间的**函数最大命中数**
   4. 使用**滑动窗口**判断函数边界
      - `j` 拓展窗口，并判断是否发生地址越界；`i` 缩小窗口
      1. 计算窗口内的二进制源函数对
      2. 计算窗口内的源函数命中数
      - 区别：1 中可能含有重复的二进制**（1-to-n 问题）**，2 中的二进制是**唯一的**
   5. 滑动窗口调整
   6. 返回区间
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img-5.png>)
5. 通过**函数调用图**校正函数对，如果 `bin1` 调用 `bin2`，则 `src1` 也调用 `src2`
   - 文章中指出通过这个调用图可以看出**三组配对关系**
   - `FUN_00028061` 和 `cJSON_AddTrueToObject`
   - `FUN_000287b3` 和 `cJSON_CreateTrue`
   - `FUN_00025679` 和 `cJSON_Delete`
- 可以参考


## 3.4. Third-party Library Detection
1. 源代码匹配二进制代码 TPL 的过程
2. 内部代码克隆问题
   1. 函数的最早发布时间
   2. 路径信息


# 4. EVALUATION
## 4.3. Results and Analysis
### 4.3.1. RQ1: Effectiveness of Function Embedding
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img-6.png>)
1. `BinaryAi` 的性能优于 `CodeCMR`，基于 `Recall@K` 和 `Top@K`
2. 消融实验证明 `CLIP` 和 `LLM` 的优越性
3. **基于嵌入向量的方法显著优秀于传统方法**，即 `B2SFinder` 和 `BinPro`


### 4.3.2. RQ2: Accuracy of Binary Source Code Matching
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img-7.png>)
1. 虽然 `BinaryAI` 的性能显著优秀于现有方法，但是它的 `Recall@1` 仍然偏低，无法实际应用，但是 `Recall@10` 的性能具有高准去绿
2. 模糊匹配情况下，平均准确率为 95%
![alt text](<images/2024_A_ICSE_BinaryAI: Binary Software Composition Analysis via Intelligent Binary Source Code Matching/img-8.png>)
3. 基于**局部性的匹配**能够显著提升二进制源代码匹配的准确性


### 4.3.3. RQ3: Accuracy of TPL Detection
BinaryAI 在 TPL 检测的表现上显著优于其他现有的 SCA 工具，尤其是在精确率和召回率方面


# RELATED KNOWLEDGE
## 1. 条件编译导致不同版本的二进制文件
1. 平台
```cpp
#ifdef WINDOWS
void myFunction() {
    // Windows 特定实现
}
#else
void myFunction() {
    // 其他平台实现
}
#endif
```
2. 编译选项
```cpp
#ifdef DEBUG
void debugFunction() {
    // 调试代码
}
#endif
```
...