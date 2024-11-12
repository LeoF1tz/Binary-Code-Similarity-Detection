| **Title** | Binary code similarity analysis  based on naming function  and common vector space |
|----------|-------------|
| **Author** | Bing Xia |
| **Institution** | State Key Laboratory of Mathermatical Engineering and Advanced Computing |
| **Venu** | Scientific Reports |
| **Year** | 2023 |
| **Paper** | [link](https://doi.org/10.1038/s41598-023-42769-9) |


# ABSTRACT
1. 创新点
   1. 通过提升不同平台的指令至**相同语义空间**，屏蔽底层指令差异
   2. 利用**图嵌入技术**学习邻居节点的稳定性语义，以保留控制流图中的相似性信息
   3. 提取**命名函数**中的特征，命名函数知识作为一种稳定的、平台无关的API信息，能更好地表达函数的语义
2. 实验结果
   - 多个维度均优于基准模型


# FUTURE WORK
1. 基于 AST
2. Reduced CFG >> **时间属性**和**文本数据**
   1. 时间属性：调用的时间或者执行顺序
   2. 文本数据：携带的文本信息、代码片段，API 或者变量名
3. 源代码摘要
4. 变量名称和变量类型的语义表达


# 1. INTRODUCTION
1. 现有挑战
   1. 跨平台的指令差异大
   2. 跨平台的控制流程图差异大
   3. 没有充分考虑不随平台改变的语义，即**函数名称**，其是由人类赋予函数功能的概括性表达
   4. 由 3 提出 ACSG 的缺点 **ACSG 见 [1]**
      1. 参数识别不准确，图中 `unknown` 标记
      2. 仅支持单平台 `arm64`
![alt text](<images/2023_3_Binary code similarity analysis  based on naming function  and common vector space/img.png>) 
2. 命名函数的优势
   - `API` 具有**平台无关性**
   - 命名函数的稳定性直接影响函数相似度的衡量
     - 如果两个二进制函数**从同一个源代码编译**而来，那么它们的命名函数（即函数调用的API序列）应该**相对相似**
   - 具有抗混淆的作用
   - 部分解决深度学习的可解释性
3. 本文方法（与挑战一一对应）
   1. 公共向量空间：将不同架构的指令映射到相同的向量空间
   2. 图嵌入
      1. 基本块为句子，其指令为单词，使用自注意力机制学习生成**基本块嵌入**
      2. 基本块嵌入 + CFG邻接矩阵 >> CFG 的图嵌入
   3. 命名空间嵌入
      - 二进制代码的 `API序列` 作为命名函数 
      - 将 `API` 进行切分，子令牌作为词，整个 `API` 为句子，再用自注意力机制学习生成函数嵌入
4. 主要贡献
   1. **`N_Match`** 结合图嵌入和命名函数嵌入，有效解决了**混架构**下二进制代码相似性问题
   2. 一种**命名函数嵌入方法**，并构建了预训练 **k2v**
   3. 实验证明 `N_Match` 的有效性，并开源



# 3. MODEL OVERVIEW
## 3.1. Framework
1. 图嵌入 $F^{\to}_g$ 
   1. 通过预训练的 `x2v` 词嵌入字典将**指令映射为向量**
   2. 使用 `LSTM` 和 `自注意力机制` 生成**基本块的向量**
   3. 利用 `Structure2vec` **基于基本块和 CFG 的邻接矩阵**生成整个 **CFG 的图嵌入**
2. 命名函数嵌入 $F^{\to}_k$
   - 基于预训练的 `k2v` 字典利，用 `LSTM` 和 `自注意力机制` 将命名函数序列转换为嵌入向量
3. 最终嵌入 $F^{\to}_{final}$
   - $F^{\to}_{final}=concat(F^{\to}_g,F^{\to}_k$ 


## 3.2. Evaluation criteria
~~一些评价指标和定义~~


## 3.3. `N_Match` implement
### 3.3.1. Common vector space embedding
1. 核心思想
   - 预训练一个模型，将不同平台的指令映射到一个公共向量空间，即 `x2v`
2. 具体方法（**值得参考**
   - 防止 **OOV 问题**出现，进行**归一化**
   1. 平台类型
   2. 内存地址
   3. 立即数替换（超过一定范围
   4. 未知指令 `UNK`
   - i.e., 
   - `mov eax, [0x3456789]` 在 `x86` 上
   - 替换为 `X_mov_eax,_MEM`


### 3.3.2. Basic block embedding
1. 核心思想
   - **将基本块节点转换为可用于图嵌入的特征向量**
   - `LSTM` 擅长处理序列数据
   - `自注意力机制` 可以聚焦于基本块中最重要的指令 
2. 具体步骤
   1. 基本块中指令序列长度标准化：过长截断，过短填充
   2. 指令嵌入：每条指令基于 `x2v` 生成嵌入向量
   3. `LSTM` >> `TRANSFORMER` >> `基本块嵌入`


### 3.3.3. Graph embedding
1. 核心思想
   - 使用 `Structure2Vec` 模型将每个节点的特征**与其邻居节点聚合**，直至生成**图级别的嵌入向量**
2. 具体步骤
   1. 图中基本块数量标准化为 `max_nodes`：过长截断，过短填充
   2. 节点初始化
   3. 信息交换：节点特征与邻接矩阵相乘
      - 邻接矩阵 `adj` 表示节点间的连接关系
   4. 节点更新和迭代
   5. 图嵌入生成

### 3.3.4. Naming function embedding
1. 核心思想
   - 提取和分割 `API` 使其成为一个有序的序列
2. 具体步骤
   1. b`API` 处理
      1. 可提取 `API`：使用 `分割` 避免 OOV
         1. 驼峰 `flagWithResult` >> `flag`, `With`, `Result`
         2. 下划线 `Set_Flag` >> `Set`, `Flag`
      2. 无法提取信息：`noapi` 标注
   2. 将处理后的 `API` 序列输入 `LSTM` 形成嵌入向量


### 3.3.5. Siamese network
1. 核心思想
   - 基于**共享参数**的孪生网络，输入一对函数，输出它们的相似值
2. ~~常见计算相似性的神经网络结构~~


# RELATED KNOWLEDGE
## 1. 增强调用点图（Augmented Call Sites Graph, ACSG）
1. 定义
   - 利用代码调用的 `API` 信息，恢复部分二进制代码的语义信息
   - 本文中利用的是**函数名称**
2. 组成
   - 每一个节点称为调用点 `Callsite`，其中，$callsite=st_1,st_2,\cdots,arg_1,arg_s$
   - $st$ `API` 名称的**子令牌**，例如 `socket`
   - $arg$ `API` 调用时所使用的**参数**，例如调用 `socket(arg, 1, 0)` 时的 `arg`, `1` 和 `0`
![alt text](<images/2023_3_Binary code similarity analysis  based on naming function  and common vector space/img-1.png>)
3. i.e.,
   1. 图中包含多个**调用点**，i.e., `socket`,  `setsocket`, `connect`
   2. 紫色箭头和绿色箭头表示对应的控制流