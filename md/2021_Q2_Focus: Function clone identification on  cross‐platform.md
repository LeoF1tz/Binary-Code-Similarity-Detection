| **Title** | Focus: Function clone identification on  cross‐platform |
|----------|-------------------------------------------------------------------------------------|
| **Author** | Lirong Fu |
| **Institution** | Zhejiang University |
| **Venue** | International Journal of Intelligent Systems |
| **Year** | 2021 |
| **Paper** | [link](https://doi.org/10.1002/int.22752) |


# ABSTRACT
1. 本文方法
   1. 采用多头注意力机制来捕捉函数的关键特征
2. 实验结果
   1. 可应用与跨架构 >> 八个架构
   2. 可应用与漏洞搜索 >> 24/30


# 1. INTRODUCTION
1. 现有挑战
   1. 现有模型无法聚焦于最重要的特征
   2. 二进制代码的语义信息难以表示
2. 贡献点
   1. `指令嵌入` + `多头注意力机制` >> `Focus`\
   2. 图注意力网络 `GTN`
   3. 开源


# 2. MOTIVATION
## 2.1. Representation of binary code
![Fig.2](<images/2021_Q2_Focus: Function clone identification on  cross‐platform/img.png>)
![Fig.3](<images/2021_Q2_Focus: Function clone identification on  cross‐platform/img-1.png>)
![Fig.4](<images/2021_Q2_Focus: Function clone identification on  cross‐platform/img-2.png>)
- `MF` 和 `CFG` 的缺陷
   1. 存在不相似函数拥有相同的 `MF` 和 `CFG` >> `Gemini` 误判为相似 i.e., `Fig.2` 和 `Fig.3`
   2. `Foucus` 使用**不同的上下文**解决这个问题


## 2.2. Differentiation of features
- **不同节点的权重不应相同** >> `Attention`


# 4. DESIGN OF `FOCUS`
![Overview](<images/2021_Q2_Focus: Function clone identification on  cross‐platform/img-3.png>)

## 4.1. Feature Engineering
### 4.1.1. Instruction normalization
- 指令标准化 >> 防止 `OOV`
  1. 寄存器地址
  2. 内存地址
  3. 立即数
  4. 相对基地址索引
  5. 基地址索引


### 4.1.2. Instruction embedding extraction
- 指令嵌入步骤
   1. 初始化 >> 随机向量
   2. 使用**滑动窗口**学习**上下文指令流**
   3. 计算优化策略
      1. 负采样
      2. 分层 softmax


## 4.2. Graph embedding
### 4.2.2. Semantic‐aware GTN
1. 核心思想
   - 使用 `多头注意力机制` 获得 $x_i$ 对 $x_j$ 的重要性
2. 具体步骤
   1. 单头注意力
      1. 线性变换 >> 使用**权重矩阵** $W$ 将每个图的节点的特征向量映射到**新的向量空间**
      2. 相邻节点重要性计算 >> $c_{ij}$
      3. 归一化 >> $softmax$
   2. 多头注意力
   3. 全图嵌入形成


## 4.3. Siamese neural network
- 使用孪生网络计算两个函数的相似性