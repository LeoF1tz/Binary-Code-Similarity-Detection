| **Title** | PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks |
|----------|-------------|
| **Author** | Xi Xu |
| **Institution** | Xi'an Jiaotong University |
| **Venu** | TSE |
| **Year** | 2023 |
| **Paper** | [link](https://doi.org/10.1109/TSE.2023.3332732) |


# ABSTRACT
1. `PatchDiscovery` 是一种提取补丁和漏洞的**关键基本块**作为**签名**的补丁存在性测试方法
   - 针对二进制函数
   - 基于一手攻击函数和已修复函数的 CFG
2. 方法
   1. 补丁定位：对易受攻击函数（VF）和1已修复函数（PF）进行预处理，生成控制流程图，并进行**标准化和简化**
   2. 补丁分析：提取并比较补丁路径和漏洞路径，评估每个块的变化程度，并根据这些变化提取**关键基本块作为签名**
   3. 补丁检测：在目标函数中分别搜索 VF 和 PF 的关键基本块，计算它们之间的相似性得分，基于此判断是否包含补丁
3. 实验结果
   1. F1 = 92.2%
   2. 平均测试一个目标函数需要 0.091s
   3. 对版本差异、函数大小和补丁大小具有鲁棒性


# CONCUSION
- 同上


# DISCUSSION
1. 函数内联使得关键基本块的识别变得困难
2. 不同的编译选项会使得补丁发生其他方面的变化。将二进制文件提升为中间表示，以实现归一化


# 1. INTRODUCTION
- 贡献点
  - 1. 补丁存在性检测：使用**关键块作为签名**识别漏洞
  - 2. 基本块匹配：通过消除语法差异并立功 CFG 的上下文信息识别范围
  - 3. 关键基本块提取：通过细粒度分析，提取关键基本块
  - 4. 简化检测流程：通过缩小搜索范围，加快检测速度
  - 5. 实验证明有效性
  - 6. 开源


# 3. OVERVIEW
## 3.1. Motivating Example
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img.png>)
1. 红色标记表示修复漏洞的基本块变化
   - (a) (c) 为漏洞函数， (b) 为修复函数
2. 关键块的重要性排序
   - `PatchDiscovery` 基于对基本块内的**指令改变数**衡量基本块的重要性，并将关键块作为签名
3. 搜索范围确定
   -  用目标函数中指向漏洞函数和修补函数的**未变化基本块作为边界指示器**


## 3.2. PatchDiscovery Overview
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-1.png>)
1. 补丁定位
   1. 对每个输入函数进行预处理，将其转化为 CFG，再进行标准化和简化
   2. 使用基本块匹配方法识别 PF 和 VF 的补丁范围和漏洞范围
2. 补丁分析
   1. 提取补丁路径和漏洞路径，基于路径评估**每个基本块变化程度**
   2. 根据变化程度对基本块进行排序，并选择关键基本块作为签名
3. 补丁存在性检测
   - 分别在TF中搜索 PF 和 VF 的关键基本块，以计算 PF 与 TF 之间及 VF 与 TF 之间的相似度分数，进而判断是否修复


## 3.3. PatchDiscovery Usage Scenarios
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-2.png>)


# 4. PATCHDISCOVERY DESIGN
## 4.1. Patch Location
### 4.1.1. Preprocessing
1. CFG 提取
2. 指令标准化和简化
3. 操作数标准化：寄存器 >> `reg`，地址 >> `addr`，内存 >> `mem`
4. 指令移除：`jmp` 和 `nop`，即无条件跳转指令和空操作指令


### 4.1.2. Basic Block Matching
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-3.png>)
1. 匹配单位：基本块
2. 基本块 `b_A` 和基本块 `b_a` 视为匹配的条件
   1. 具有相同的归一化指令序列
   2. 具有相同的归一化指令序列，除了**语法不同但语义等价**的条件跳转指令
      - 如图`4`
      -  (a) 中是 `jne J_addr`，表示条件不满足时，跳转到 `J`
      -  (b) 中是 `je q_addr`，表示条件满足时，跳转到 `q`，**即条件不满足时，跳转到 `j`**
      -  两者是语法不同但是语义等价的条件跳转指令
3. 特征基本块标记：优先级顺序排列，一个块可以满足多种特征，**但是只能属于一种类别**
   1. 入口块：函数开始执行的基本块
   2. 出口块：函数结束执行的基本块
   3. 字符串引用块：包含至少一个字符串引用指令的基本块
   4. 函数调用块：包含至少一次函数调用的基本块
4. 匹配搜索：利用**前后文关系**寻找匹配块
   1. 目的：根据**已识别的 PF 中的最高级基本块 `b_B`** 在 VF 中寻找到与之对应的 `b_b`
   2. 方法：
      1. 前驱匹配
         - 对于 `b_B` 的前驱块 `b_A`，如果 `VF` 中有对应的 `b_a`，则将**其未匹配的所有后续块**加入到集合 `B_F` 中
      2. 后继匹配
         - 对于 `b_B` 的后继块 `b_C`，如果 `VF` 中有对应的 `b_c`，则将**其未匹配的所有前驱块**加入到集合 `B_N` 中
      3. 确定候选集合 `B`
         1. `B_F` 和 `B_N` 非空，取两者**交集**为候选集合
         2. 否则，取两者**并集**为候选集合 
      4. 匹配
         - `b_B` 与候选集合 `B` 中每一块进行匹配



## 4.2. Patch Analysis
### 4.2.1. Path Extraction
- 对于 `VF` 和 `PF` 都使用路径提取，提取出漏洞路径集合 `PV`
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-4.png>)
1. 遍历属于补丁基本块集合 $B_p$ 的每一个基本块
2. 如果当前 $b$ 为入口块或 $b$ 的前驱集合跟属于 $BB_{PV}$ ，即 $PF$ 和 $VF$ 的已匹配基本块集合
   1. 该块作为路径起始块
   2. 调用 `EXTRACT` 函数确定之后的路径
3. 返回路径


![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-5.png>)
1. 如果该基本块 $b$ 属于补丁基本块，加入路径
2. 如果该块属于三个终止条件之一，则将该路径加入到路径集合中，并结束
   1. $b\notin B_p$ ，即 $b$ 超出了补丁块范围
   2. $b$ 没有后继块，即 $b$ 是函数终止块
   3. 路径出现循环
3. 不满足终止，则对后继块递归调用 `EXTRACT`，确定之后的路径


### 4.2.2. Key Basic Block Selection
1. 确定关键基本块
   - 通过计基本块中**更改的指令数**确定关键基本块
2. 建立补丁路径和漏洞路径的对应关系
   1. 粗略匹配
      - 目的：排除不相关路径
      - 方法： 补丁路径 `pP` 的邻近基本块和漏洞路径 `pV` 的邻近基本块之间存在匹配关系，则 `pV` 会被标记为 `pP` 的候选路径
   2. 精确匹配
      - 目的：选择与补丁路径 `pP` 具有最高**相似性**的漏洞路径 `pV` 进行变化度计算
      - 方法： 使用 `LSH FOREST` 进行路径相似度计算
   3. 变化度计算
      1. 使用 `LCS` 计算最长公共子序列，其反应了两条路径之间的最常相似性片段，并使用**不在 `LCS` 的指令数作为变化度**
      2. 对于重复路径，算法算去所有变化都中最小值作为该块的最终变化度
   4. 关键块选择和排名
      1. 根据变化度排名，分别选择 **`Top-N` 关键基本块**，即补丁签名（KBP）和漏洞签名（KBV）
      2. 目的：找出最能代表补丁和漏洞变化的关键基本块


## 4.3. Patch Presence Discovery
### 4.3.1. Search Range Reduction
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-6.png>)
1. 目标：针对关键基本块，在 `TF` 中确立一个更小的搜索范围
2. 方法：i.e., `PF`
   1. 使用集合 `BB_pt`，即 `PF` 和 `TF` 中已匹配的基本块集合，将 `CFG` 划分为 `sub_CFG`
   2. 使用 `PF` 的基本块 `kbP` 所对应的 `sub_CFG` 匹配 `TF`相似的 `sub_CFG`
   3. 在 (2) 中的 `sub_CFG` 中搜索 `kbP` 对应的 `TF` 的基本块


### 4.3.2. Similarity Calculation
- `PF` 中补丁导致新增基本块
   1. `TF` 的 `sub_CFG` 中不具有对应块，则该 `TF` 对应为 `VF`，即补丁未修复，为漏洞函数
   2. 否则，计算两个函数中的关键基本块的相似性表示这两个函数的相似性
      1. 对于关键块 `kb` 和 TF 的基本块 `b` 使用 `LCS` 计算相似性
      2. 基于**关键块权重**和该块的相似性计算整个函数的相似性
      3. 如果 $SIM(PF,TF) > SIM(VF, TF)$ ，则认为已修复


# 5. EVALUATION
## 5.1. Answer to RQ 1: Effectiveness and Efficiency
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-10.png>)
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-11.png>)
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-12.png>)
1. 失败原因
   1. 改动非常小，例如关键块中仅有两条指令，其中只有一条指令发生改变
   2. 改动非常大，无法找到匹配的基本块
2. 确定补丁范围和漏洞范围可以有效减少分析规模，并保持足够的细粒度




## 5.2. Answer to RQ 2: Resilience to Version Gap
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-9.png>)
1. 相近版本的效果更好
2. 随着版本差异的增加，代码的变化程度也增加；版本差异较大时假阴性减少


## 5.3. Answer to RQ 3: Resilience to Function Size and Patch Size
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-7.png>)
1. `PatchDiscovery` 针对的是**补丁级别**，而不是函数级别，故函数大小对 `PatchDiscovery` 的影响不大
![alt text](<images/2023_A_TSE_PatchDiscovery: Patch Presence Test for Identifying  Binary Vulnerabilities Based on Key Basic Blocks/img-8.png>)
2. 补丁过小时，补丁引入的修改可能在目标函数中消失或者融合（因编译）


# 6. DISCUSSION
## 6.1. Function Inlining
1. 影响
   1. 补丁无关的代码改动
   2. 关键基本块搜索失败
2. 潜在方法
   - `Asm2Vec` 选择性内联


## 6.2.  Compilation Differences
1. 影响
   - 不同的编译优化级别可能会改变二进制代码的结构
2. 潜在方法
   - 使用中间表示转换为嵌入向量