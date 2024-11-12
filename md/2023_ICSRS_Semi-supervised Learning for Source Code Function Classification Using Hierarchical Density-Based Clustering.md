| **Title** | Semi-supervised Learning for Source Code Function  Classification Using Hierarchical Density-Based  Clustering |
|----------|-------------|
| **Author** | Zhi Sun |
| **Institution** | The 30th Research Institute of China Electronics Technology Group Corporation |
| **Venu** | ICSRS |
| **Year** | 2023 |


# ABSTRACT
- 本文方法：
  1. 目的：源代码主题分类
  2. 方法：使用**半监督学习技术**，即有限的标注数据和大量**未标注数据**相结合，并利用**分层密度聚类**


# CONCLUSION
1. 研究问题：自动化开源代码功能分类的挑战
   1. 标注函数的稀缺性
   2. 代码语义特征的自动聚类
2. 方法创新点
   1. 深度领域自适应：模型**自适应地理解不同功能**代码的特征
   2. 自动特征学习：使用**半监督的分层密度聚类算法**，将相似的代码构件归类成组
3. 实验结果
   - 在十个实际软件项目上测试，证明其有效性
4. 未来方向
   1. 减少预处理时间
   2. 增加代码分类的多样性
   3. 提高分类准确性


# 3. PROPOSED APPROACH
## 3.1. Extract the Source Code Snippet and Description




## 3.2. Unsupervised Clustering of Code Description Information
1. 使用 `paraphrase-multilingual-miniLM-L12-v2` 预训练模型将**代码描述信息转**换为向量
2. 使用 `HDBSCAN` 作为聚类算法
   1. 将代码描述信息的向量转换为**距离矩阵[1]**
      - 构建每个点之间的成对距离
   2. 基于距离矩阵创建**最小生成树[2]**
      - 根据距离确定边的权重
   3. 定义每个点的**核心距离[3]**
      - 核心距离指的是到第 k 个最近邻的距离
   4. 定义互达距离
      - 即两点之间的最大核心距离和实际距离的最大值
   5. 最该最小生成数中的边权重
      - 两点距离 >> 互达距离
   6.  应用单链接聚类构建分层聚类模型
       - 将互达距离最小的两个簇合并 
   7. 分层聚类 >> 平面聚类


## 3.3. Cluster Topic Correlations and Build a Labeled Dataset
1. 确定代码功能类别
   - 网络、系统、硬件、身份验证和时间
2. 使用 **`TF-IDF`[4]** 提取每个**聚类的关键词**
   - 关键词代表了源代码的语义功能
3. 计算与预设类别的相似性
   - 使用 `paraphrase-multilingual-MiniLM-L12-v2` 模型，计算每个聚类和预设的五个类别之间的语义距离。低于阈值，则视为同一类
4. 手动修正 (3) 生成的类别标签


## 3.4. Fine-Tune the Source Code Classification Model
1. 预处理
   - 将数据集转换为 `UniXcoder` 可以处理的格式
   1. 使用 `RoBERTa` 分词器对代码进行分词
   2. 对 `AST` 进行编码 
2. 设定模型
   1. 冻结最后两层
   2. 添加一个 `6 维` 的全连接层


# 4. EVALUATION
![alt text](<images/2023_ICSRS_Semi-supervised Learning for Source Code Function Classification Using Hierarchical Density-Based Clustering/img.png>)
![alt text](<images/2023_ICSRS_Semi-supervised Learning for Source Code Function Classification Using Hierarchical Density-Based Clustering/img-1.png>)
![alt text](<images/2023_ICSRS_Semi-supervised Learning for Source Code Function Classification Using Hierarchical Density-Based Clustering/img-2.png>)
![alt text](<images/2023_ICSRS_Semi-supervised Learning for Source Code Function Classification Using Hierarchical Density-Based Clustering/img-3.png>)
1. 数据集构建与标注
   1. 使用半监督的 `HDBSCAN` 方法对 **1,602,343 个代码函数**进行了主题聚类
   2. 使用基于类别的 `TF-IDF` 方法提取每个类别的代表性关键词，将阈值低于0.3的样本纳入训练数据集
   3. 人工标注修正 （2）
2. 模型性能比较
   - `UniXcoder` 最好
3. 概念存在交叉的分类任务表现较差：**硬件和系统**



# RELATED KNOWLEDGE
## 1. 距离矩阵
\[
\begin{bmatrix}
0 & d_{AB} & d_{AC} \\
d_{BA} & 0 & d_{BC} \\
d_{CA} & d_{CB} & 0 \\
\end{bmatrix}
\] 其中 $A, B, C$ 分别表示代码，$d$ 表示向量的距离

## 2. 最小生成数（prim、Kruskal）
i.e., 基于 `Kruskal`
\[
\begin{bmatrix}
0 & 2 & 3 \\
2 & 0 & 1 \\
3 & 1 & 0 \\
\end{bmatrix}
\]
1. 按距离选择边：1 >> 2 >> 3
2. 构建生成树：B-C >> A-B，即结构为 $d_{B-C}=1, d_{A-B}=2$


## 3. 核心距离
1. 定义
   - 一个点与它的**第 k 个最近邻点**之间的距离。它用于衡量该点在**其邻域中的密度**。理解核心距离有助于**判定点在数据集中是否处于密度较高**的区域
2. i.e.,
   - 点 `P` 的核心距离为离 `k`，即 `P` 点的第 `k` 个最近邻的距离为 `k`
   - 与阈值相区别，核心距离是**动态变化的**，每个点的邻域中**自适应密度**


## 4. TF-IDF
1. 定义
   - 结合某词在文档内出现的**频率和分布情况**衡量其重要性
2. 计算原理
   1. `TF`
      1. 定义：特定词在**当前文档**出现的频率，即**词频**
      2. 公式：$TF=\frac{该词在当前文档中出现的次数}{当前文档的总次数}$
   2. `IDF`
      1. 定义：一个词在**整个文档集中**的普遍性，即逆文档频率
      2. 公式：$IDF=log\frac{文档总数}{包含该词的文档数}$
   3. `TF-ID` 
      - 公式：$TF-IDF=TF\times IDF$