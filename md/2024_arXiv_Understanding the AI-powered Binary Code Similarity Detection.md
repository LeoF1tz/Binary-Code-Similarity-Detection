| **Title** | Understanding the AI-powered Binary Code Similarity Detection |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Lirong Fu |
| **Institution** | Hangzhou Dianzi University |
| **Venue** | arXiv |
| **Year** | 2024 |

# ABSTRACT && CONCLUSION
1. GNN 有很大的提升空间 >> 图对齐等
2. 不用场景的性能测试 >> 不同 BCSD 方法在特定的下游任务中性能差异显著
3. 评估常用 `Metric` >> 某些评价指标无法准确评估模型性能
4. 开源


# FUTURE WORK
1. LLM to BCSD
2. 不同类型的嵌入进行拼接
3. 图对齐 >> 比较基本块的属性


# 1. INTRODUCTION
1. 现有不足
   1. 不同的模型性能差异难以量化
   2. 现有模型缺乏解释性
   3. 现有评价指标无法准确评估模型性能
   4. 对下游应方法缺乏测试 >> 漏洞检测
2. 本文贡献
   1. 对现有模型性能进行量化且公平的检测
   2. 检测当前模型对下游任务的执行能力


# 2. BACKGROUND
## 2.1. Workflow of AI-powered BinSD
1. 预处理：提取代码的特征
   - 二进制 >> 反汇编 >> 指令规范化（防止 OOV）
     - 模拟内联
     - 手动特征提取
     - 指令嵌入
2. 代码表示：表示二进制代码的方法 
   - `AST`, `CFG`, `DFG`, 指令执行序列，（ `IR`
3. 代码嵌入：将代码表示转换为嵌入向量
   - `GNN`, `CNN`, `LSTM`, `BERT`


## 2.2. Paper Selection
1. 基于**静态**分析
2. 基于**函数级**对比
3. 不涉及抗混淆 >> 非主流


# 3. HOW BINSD APPROACHES PERFORM
## 3.3. Similar Function Detection
![alt text](<images/2024_arXiv_Understanding the AI-powered Binary Code Similarity Detection/img-2.png>)
### 3.3.1. Accuracy Evaluation
1. 跨架构，跨优化等级
2. 现有问题
   1. **函数重命名** >> 存在函数名字相同，但是实际功能不相似
   2. 存储库构建 >> 数据集中和查询函数可能相似，也可能不相似，导致性能差异


#### 3.3.1.1. Cross-optimization level and architecture evaluation
1. 不同的 `BINSD` 在不同的场景下表现排名不一，即每个 `BINSD` 的擅长领域不同
2. 基于 `GNN` 的方法表现相对优秀
3. 现有指标并不能很好的反应不同 `BINSD` 的性能差异
   - `AUC`, `RECALL`, `MAP`, `MAP`, etc.
4. 基于指令嵌入 >> 基于手动特征
5. **对不同架构下编译的二进制代码不够鲁棒**


#### 3.3.1.2， The influence of function rename
- 函数重命名的影响总体较小


#### 3.3.1.3. The influence of repository construction
![alt text](<images/2024_arXiv_Understanding the AI-powered Binary Code Similarity Detection/img-3.png>)
1. 实验设置
   1. 移除训练集中与查询函数相同的函数
   2. 控制存储库中查询函数的比例
2. 实验结果
   - 泛化能力较差
   2. 移除与查询函数相同的函数后，各项指标显著下降，即模型性能下降
   3. 控制查询函数的比例，反映出模型性能和库中存储函数比例占比呈正相关


### 3.3.2. Efficiency Comparision
1. 不同 `BINSD` 的训练时间和**嵌入生成时间**差异很大
2. 模型训练是一次性过程，应该更关注于**嵌入生成的时间**


## 3.4. Downstream Application
### 3.4.1. Vulnerability Search
1. 实验目的：识别相似漏洞函数
2. 实验结论
   1. 误报率较高 >> 假阳性多
   2. 使用固定阈值判断漏洞不可行
      1. 该漏洞的真实函数可能具有**较宽的分数分布范围** 
         - `CVE-2014-0195` 中，漏洞的最大和最小相似度分别为 `0.93` 和 `0.8`
         - `CVE-2014-3513` 中，所有搜索结果都大于 `0.9`，但是**并没有一个是漏洞**
   3. **过高阈值，遗漏真实漏洞；过低阈值，引入大量误报**
- ~~自适应阈值？~~


### 3.4.2. License Violation Detection
1. 实验目的：识别第三方库
2. 实验结论
   1. 该任务下的**存储库较小**，此时适用邻接矩阵描述 `CFG` 的 `BINSD` 方法
   2. 与漏洞检测相比，许可证违规检测准确性更高 >> 漏洞检测更具有挑战性


# 4. UNDERSTANDING OF EMBEDDING NETWORKS AND EVALUATION METRICS
## 4.1. Embedding Neural Networks
### 4.1.1. RNN
1. 方法：通过聚合邻居节点的方式最终得到整个图的特征向量
2. 使用场景：同构图 >> 对比图的**拓扑结构和顶点数量**
3. 存在问题
   1. 图发生改变 >> 跨架构和函数内联
   2. 嵌入冲突 >> 最终图的聚合方式是使用 `READOUT` 函数，其是通过**加法实现**，即可能会出现**不同节点的嵌入和刚好相等** >> 误判为相似（FPs 


### 4.1.2. CNN
- 存在问题：可学习特征较少
  1. 每个二进制函数中的基本块普遍较少 >> $Basic Block <= 100$
  2. 基本块中的关系普遍简单


## 4.2. Evaluation Metrics
![alt text](<images/2024_arXiv_Understanding the AI-powered Binary Code Similarity Detection/img.png>)
1. `AUC`
   1. 定义
      - AUC 越高，模型性能越好；即模型具有一个良好的区分阈值来确定真阳性和真阴性，并且保持较低的假阴性和假阳性
   2. 局限性：`BIN` 可能存在 `AUC` 高，但**实际性能较低**的情况，即具有迷惑性
      - 在 `BIN` 模型进行测试时，通常会从测试集中**随机选择**两个样本
      - 负样本情况下，它们的特征往往存在显著差异 i.e., `CFG`, `Function`，这就从客观上造成它们**本来就易于被正确区分**
   3. 通过图中的 (a) (b) (c) 可以观察到，模型输出的结果本来就**具有某种程度的两极分化**，即找到一个阈值是相对容易的
![alt text](<images/2024_arXiv_Understanding the AI-powered Binary Code Similarity Detection/img-1.png>)
2. `Recall@K`
   1. 定义：从查询结果中选取前 `K` 个结果时，系统成功检索出的相关结果所占的比例
   2. 作者认为 `BIN` 对二进制函数的相似性检测更类似**推荐**，即需要对大量函数进行比较，并从中找出**与查询函数最相似的前 `K` 个函数**，这样更具有挑战性