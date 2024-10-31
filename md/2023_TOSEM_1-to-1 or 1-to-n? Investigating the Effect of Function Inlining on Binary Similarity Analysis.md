| **Title** | 1-to-1 or 1-to-n? Investigating the Effect of Function Inlining on Binary Similarity Analysis |
|----------|-------------------------------------------------------------------------------------|
| **Author** | ANG JIA |
| **Institution** | Xi’an Jiaotong University |
| **Venue** | TOSEM |
| **Year** | 2023 |

---

# ABSTRACT
![alt text](<images/1-to-1 or 1-to-n/img.png>)
- 1-to-1
  - 一个二进制函数和一个二进制/源代码函数进行匹配
  - 目前大多数工作中采用的方式
- 1-to-n / n-to-n
  - 由于**函数内联**的存在，实际上是 **1-to-n / n-to-n**
  - 1-to-n
    - 一个二进制函数匹配多个源函数或二进制函数
  - n-to-n
    - 多个二进制函数匹配多个二进制函数
- 本文工作
  - 调查函数内联的程度
    - 在 O3 优化下时，函数内联比例在 30%-40%，最高甚至到 70%
  - 函数内联对现有方法的影响
    - 函数搜索下降30%，漏洞搜索下降40%
  - 现有函数内联模拟策略的有效性
    - 40% 内联没有被识别到
    - 建议使用条件和增量的内联策略



# CONCLUSION
1. **首次**研究**函数内联**对 BCSD 的影响
   - 1. 函数内联广泛存在，并且和优化等级呈正相关，出现频率为 36-70%
   - 2. BCSD 的搜索性能下降 30%，漏洞检测下降 40%
2. 提出关于内联模拟的**建议**




# 1. Introduction
- 传统 BCSD 方法假设 1-to-1，但实际上由于函数内联的存在，可能是 1-to-n 甚至是 n-to-n
- Figure 1
  - (a) 表示漏洞函数 `scm_send` 的源代码
  - (b) 表示 `scm_send` 的 CFG
  - (c) 表示 `netlink_sendmsg` 的 CFG
    - `netlink_sendmsg` 使用了**函数内联**，因此 `scm_send` 此时作为 `netlink_sendmsg` 的 CFG 的一部分
  - (d) 表示 `netlink_sendmsg` 和 `scm_send` 的函数内联
    - `scm_send` 被内联到了 `netlink_sndmsg` 中
    - `scm_set_cred` 也被内联到了 `scm_send` 中，形成了**嵌套内联**
- 由于**函数内联改变了函数的结构**，简单的相似性匹配无法识别这些嵌套的漏洞函数
- 本文回答的 RQ 问题
  - 1. 函数内联的频率和程度
    - 内联比例为 40% - 70%
    - 一个二进制函数可以内联 2 - 4 个源函数
  - 2. 函数内联对 BCSD 的影响
    - 性能下降 30%
  - 3. 现有的函数内联模拟方式的有效性


# 2. Preliminary
## 2.1. Software Supply Chain Security Tasks
- 现有的软件开发高度依赖于第三方库，这样就引申出四个 BCSD 的相关工作
  - 1. 代码搜索
    - 使用 A 函数作为查询函数，**查找与 A 函数功能相同的函数**，返回与相似度的排名
    - 代表工作 `SAFE`，转换成**嵌入向量**，再基于嵌入向量之间的距离作为相似性依据
  - 2. OSS 重用检测
    - 查找已编译的二进制文件中的源代码重用
  - 3. 漏洞检测
    - 比较目标函数与漏洞函数来检测 1-day 漏洞
  - 4. 补丁存在测试
    - 当漏洞和补丁差距很小时，区分漏洞函数是否已经打了补丁


## 2.2. Research Scope
- **常见编译器**所使用的**函数内联策略**对**静态函数**级别的二进制代码相似性粉丝的影响
- 编译器种类
  - GCC / C++
- 编译优化等级
  - `O0 - O3`
- 架构选择
  - X86-64


# 3. Dataset Construction
- 自制数据集
- 提出一种可拓展且轻量化的针对二进制中内联函数的识别工具


## 3.1. Dataset Collection
- 1. 代码搜索
  - GCC, Clang
  - O0 - O3
  - X86, ARM MIPS, MIPSeb
- 2. OSS 复用
  - 基于 `ISRD` 的数据集
  - O0 - O3
  - X86-64
- 3. 漏洞检测
  - 基于 `BinXray`
  - 两个版本：使用/不使用内联
- 4. 用于应用程序二进制文件
  - 模拟大型程序中使用函数内联的场景


## 3.2. Function Inlining Identfication
### 3.2.1. Terminology
![alt text](<images/1-to-1 or 1-to-n/img-1.png>)
- BFI：包含了内联源函数的二进制函数，例如 `netlink_sendmsg`，它不仅包含自己的逻辑，还包含了内联的 `scm_send` 和 `scm_set_cred`
  - OSF：未经内联的原始函数，例如源代码中的 `netlink_sendmsg` 函数
  - ISFs：在内联过程中嵌入到其他函数中的源函数，例如 `scm_send` 和 `scm_set_cred`
- NBF: 普通二进制函数，即未发生内联的普通函数
- NSF: 普通源函数，即未发生内联的普通函数
- BS, NS：分别表示一般的二进制函数和源函数的统称


### 3.2.2. Function Inlining Identification Method
- 方法
  - 1. 编译源代码生成调试信息，
    - `-g` 在编译过程中生成调试信息 >> `.debug_line` 得到调试信息
  - 2. 提取行级映射
    -  使用 `Readelf` 提取信息 >> 得到源代码行号和二进制地址之间的映射关系
  - 3. 建立源代码行到函数的映射
    - 使用 `Understand` >> 确定函数的边界，每行代码属于哪个函数
  - 4. 建立二进制地址行到函数的映射
    - 使用 `IDA Pro` >> 确定函数的边界，每条指令属于哪个函数
  - 5. 组合映射生成**函数级别的关联**，实现**自动标记**数据集
![alt text](<images/1-to-1 or 1-to-n/img-2.png>)
- 1. 实现行级别的匹配，一个属于 287 行，一个属于二进制地址 `b2sum`
- 2. 确定函数的边界，分别都属于 `usage`
- 3. 实现函数级别的匹配，即 `usage`


### 3.2.3. Effectiveness of Function Inlining Identification Method
- 结果
  - `ISFs` 的准确率为 100%
  - 识别源代码行所属源函数的 `Recall` 为 88%
  - 二进制代码识别
    - 二进制函数 100%
    - 源代码行 88%
  - 需要手动检查


## 3.3. Application Matching Labels Considering Function Inlining
- 匹配标签（区别在于有没有将 ISF 视为函数的一部分）
  - 1. 代码搜索
    - 给定一个 query BF
      - 标记来源于同一个源函数的 target BF
      - 标记来源于同一个源函数的 target SF 
    - BF 和 ISF 不会被标记为匹配对（ISF 已经被内联到函数中，不作为函数独立存在）
  - 2. OSS 重用检测
    - 超过 90% 的相似性则认为是OSS 重用
    - 将 ISF 计算在内
  - 3. 漏洞检测
    - 与 OSS 相似，将 ISF 计算在内
  - 4. 补丁存在测试
    - - 与 OSS 相似，将 ISF 计算在内


# 4. Investigation on the extent of function Inlining
## 4.1. What is the extent of function lnlining under different compilations
![alt text](<images/1-to-1 or 1-to-n/img-3.png>)
- 优化级别
  - 随着优化级别升高，内联比率也显著升高
- 编译器
  - 同一系列的编译器在内联方面差异不大
  - GCC 在 O1 优化应用了 `-finline-functions-called-once`
  - Clang 应用 `always inliner`
- 架构
  - 64 位的内联程度大于 32 位（x86 除外）
  - 不同架构导致不同的编译路径和函数调用图，进而导致内联策略的不同
![alt text](<images/1-to-1 or 1-to-n/img-4.png>)
- 链接时优化
  - 什么是链接时优化（LTO）
    - **链接阶段**对整个程序的**所有编译单元**进行全局分析和优化
  - 与默认优化策略`Ox`的区别
    - `Ox` 的优化是在链接阶段前
- 结果
  - `LTO` 的内联比率明显提高，在 `O3` 下时，超过 70%
  - `LTO` 内联的程度提高，`BFI` 会更多的内联 `ISF`


## 4.2. What Is the Extent of Function Inlining in Different Projects?
- 1. 某些项目可能存在很长的调用链
  - 漏洞检测的数据集中一个 BFI 可能内联了 超过 2000 个 ISF，一个 ISF 可能被内联到超过 800 个 BFI 中
- 2. 在大型应用程序的二进制文件中，会使用基于 **性能分析的内联和 LTO** 导致内联程度更大
  - 基于性能分析
    - 使用性能分析工具，通过测试不同场景下不同内联方式的性能，基于执行结果的性能决定最后的内联方式
  - LTO
    - 允许跨过程的函数调用


## 4.3. What Parts of Inlining Are Cared for in Different Applications?
- 不同应用的目标不同，应该关注的内联部分也不同
- 1. 代码搜索
  - 关注点：**完整的功能等价性**，只关心是否找到功能上相同的源函数
  - Query:源函数`A`
  - Result: 功能上等同于 `A` 的函数
    - `A` 被内联到 `B`中，`B` 不需要被检测出来
- 2. OSS/漏洞/补丁
  - 不仅关注OSF，还需要**识别和匹配内联的ISFs**，以找到重用、漏洞或补丁的代码片
  - Query: `A`
  - Result: 所有包含 `A` 的函数
    - `A` 被内联到 `C`中，`C` 也需要被检测出来


# 5. PERFORMANCE OF EXISTING WORKS UNDER INLINING
- 编译成两类
  - 1. 使用内联
  - 2. 不使用内联


## 5.1. Evaluation of Code Search Works
![alt text](<images/1-to-1 or 1-to-n/img-5.png>)
- 二进制 - 源代码匹配 `CodeCMR`
  - 跨优化等级
    - (a) 红色为不使用内联，蓝色为使用内联
    - `Recall@1` 下降 30%，`Recall@10` 下降 25%
  - 内联程度评估
    - (b) 
    - 内联程度为 0（横坐标）代表没有内联
    - 内联程度为 1 代表使用了内联，且内联后的代码长度为内联前的 1-2 倍
    - 随着内联程度上升，`Recall` 指标随之下降
![alt text](<images/1-to-1 or 1-to-n/img-6.png>)
- 二进制 to 二进制 `SAFE`
  - 联匹配（蓝色）与不含内联匹配（红色）差异巨大，尤其是在优化等级高时
  - Clang（实线）的影响程度高于 GCC（虚线）
    - ~~编译器的种类是影响性能的因素之一~~


## 5.2. Evaluation of OSS Reuse Detection Works
![alt text](<images/1-to-1 or 1-to-n/img-7.png>)
- 方法 `CodeCMR`
- 1. NSF
  - 不使用内联的性能最高
- 2. OSF
  - 使用内联，但是 OSF 的核心功能和查询函数是类似的，因此，函数主体没有太大变化，性能不受太多影响
- 3. ISF
  - 查询函数作为函数的一部分，被内联到其他函数，基本无法识别出来


## 5.3. Evaluation of Vulnerability Detection Works
![alt text](<images/1-to-1 or 1-to-n/img-8.png>)
- 方法 `Vulseeker`
- 结果与 OSS 类似，搜索 NSF 和 OSF 的表现很好，但是对于 ISF 的性能显著降低


## 5.4. Evaluation of Patch Presence Test Works
![alt text](<images/1-to-1 or 1-to-n/img-9.png>)
- 内联导致识别正确率下降，误判增加，并且使得更多函数变得无法分析


## 5.5. Answering to RQ2
- 代码搜索任务
  - 内联会带来 20-30% 的性能损失
- OSS/漏洞/补丁
  - 内联会**严重影响**模型的性能，这就会导致攻击者可以将漏洞内联到其他函数中绕过检测



# 6. EVALUATION OF EXISTING INLINING-SIMULATION STRATEGIES
## 6.1. Introduction of Existing Inlining-Simulation Strategies
### 6.1.1. Bingo 内联模拟策略
- `Bingo` 的内联策略（2-5 适用于用户定义函数）
- 1. 被调用函数为库函数，则内联
- 2. UD 被调用函数与调用者递归调用彼此，则内联
- 3. UD 被调用函数仅调用库函数，且超过一半的库函数不是终止函数，则内联
- 4. UD 被调用函数仅调用库函数，且超过一半的函数是库函数，**则非内联**
- 5. UD 被调用函数调用其他 UD 函数：（根据函数调用图）
  - $\alpha (fc)=\frac{outdegree(fc)}{outdegree(fc)+indegree(fc)} $
  - $\alpha$ 越低，越容易内联；低于 $0.01$ 视为内联


### 6.1.2. ASM2Vec 内联模拟策略
- 基本与 `Bingo` 类似
- 区别
  - 1. `ASM2Vec` 使用**一层内联模拟策略**，代替 `Bingo` 的递归模拟
    - i.e., `A` 调用 `B`，`B` 调用 `C`
    - `Bingo` 将 `C` 内联到 `A`
    - `ASM2Vec` 只将 `B` 内联到 `A`
    - 目的：减少计算复杂度和内存消耗
  - 2. 过滤长函数内联
    - $\delta(f_s, f_c) =\frac{length(f_c)}{length(f_s)}$
    - $f_c$ 表示被调用函数
    - $f_s$ 表示调用函数
    - $length$ 指令长度
    - 低于 $0.6$ 内联，或者被调用函数的指令长度小于 10
    - 目的：避免将过长的函数内联进来，减少计算复杂度和内存消耗

## 6.2. Evaluating Strategies on Finding Similar BFIs
- 使用两个指标，**成本**和**相似性**，评估 `Bingo` 和 `ASM2Vec` 的内联策略在查找具有相同 **OSF** 的 **BFIs** 上的有效性
1. $cost = |BFI'_1-BF1|-|BFI'_2-BF2|$
   - 定义 
     - `BINGO` 和 `ASM2VEC` 的模拟内联策略可能会将某些**实际上未内联**的函数视为内联，导致了额外的内联操作，这也就导致了 cost 的增加，即增加了内联的复杂性和计算开销
   - 结果
   - 
2. $similarity=\frac{BFI'_1\cap BFI'2}{BFI'_1\cup BFI'_2}$
   - 尽管 cost增加，但是这种模拟内联策略也能带来正面效果，即 更高的 Similarity，进而使得模型的检测效果提升
3. 关于图 12 的内容
   ![alt text](<images/1-to-1 or 1-to-n/img-10.png>) 
   2. cost
      - `cost-WI` 可以看到不使用模拟内联的方式，并没有额外的开销
      - 在使用了**模拟内联**后，`BINGO` 的开销显著提高，其次是 `ASM2VEC`
        - `BINGO` 递归模拟
        - `ASM2VEC` 使用一层内联模拟策略作为优化方式，因此开销更低 
   3. similarity
      - `sim-WI` 的性能最低
      - `sim-Bingo` 和 `sim-ASM2VEC` 性能提高，表明这两个方法的内联模拟策略是有效的


## 6.3. Evaluating Strategies on Finding Inlined Functions
1. 目的
   - 探究 `BINGO` 和 `ASM2VEC` 在特定领域对查找函数内联的能力
2. 表 6 解释
![alt text](<images/1-to-1 or 1-to-n/img-11.png>)
   - 1. Kind 内联函数种类
     - `BFNN` 表示实际不需要被内联的函数
     - `BFN` 表示实际需要被内联的函数
   - 2. Case 针对 `BINGO` 的模拟内联条件
     - $\alpha$ 满足条件 5
     - $L$ 满足条件 3
     - $R$ 满足条件 2
   - 3. 优化等级对比
     - `O0-O1` 表示优化等级 `O0` 和优化等级 `O1` 进行匹配
   - 4. `coverage`
     - 表示当前方法**实际覆盖**了多少**实际需要**被内联的函数
   - 5. 结果
     -  `ASM2VEC` 和 `BINGO` 内联了大量实际不需要被内联的函数，尤其是那些满足其所设定的条件 3 的函数，最高甚至到了 68%
     -  在实际需要被内联的函数中，覆盖率又不足。同样是条件 3 情况下，最高 30.49% 实际需要被内联的函数没有被内联
     -  **内联覆盖率较低**，最差情况下只有 61% 实际需要被的函数被内联
     -  针对两个模型进行比较，可以看到 `ASM2VEC` 可以过滤一些 `BFNN`，但是代价是，他会额外漏掉一些 `BFN`，原因，**筛选条件过于严格**
3. 图 13 解释
![alt text](<images/1-to-1 or 1-to-n/img-12.png>) 
   - 1. i.e.,
     - `A` 内联了 `B`，`B` 内联了 `C`，`C` 内联了 `D`
     - 则它们的内联深度（Depths of BFIs）分别为 `B = 1, C = 2, D = 3` 
     - 选择**最大深度**作为最终结果 
   - 2. 结果
     - (a) 二进制函数内联深度
       - 大概 30% 的 BFI 具有超过 1 层的深度
       - `ASM2VEC` 采用一层内联策略，面对这种情况，性能可能会不佳
     - (b) 内联代码的代码行数
       - 1. 有以万计的 `ISF` 超过 100 行
       - 2. 极端情况下，最常的代码行数为**19083**，且它被长度为**不到10行**的代码内联，这种**长度悬殊**的内联方式，给现有方法造成了极大挑战
     - (c) 内联函数代码与其源函数长度比率
       - 1. 差异很大，长度不一
       - 2. `ASM2VEC` 的内联条件为内联函数和源函数的长度比例为小于 0.6 时，才会将这个内联函数视为内联候选，这就导致可能会漏选


## 6.4. Answer to RQ3
现有方法，特指 `BINGO` 和 `ASM2VEC` 能检测出部分内联函数，但是还有很大的提升空间

# 7. TOWARDS MORE EFFECTIVE STRATEGIES
## 7.1 Investigation of Matching Patterns Under Inlining
1.现有**函数级**内联方式总结：
![alt text](<images/1-to-1 or 1-to-n/img-13.png>)
- 1. 1-to-1
   - (a) 不使用内联
   - (b) 两个二进制函数内联了相同的源函数
- 2. 1-to-n
   - (c) (d) 一个二进制函数使用内联，另一个不使用内联
- 3. n-to-n
   - (e) (f) 两个二进制函数内联了不同的源函数，最困难
![alt text](<images/1-to-1 or 1-to-n/img-14.png>)
2. 内联方式的出现频率不一样，大多是 (a) 和 (c)，总计为 89%；最难的 (e) 和 (f) 出现频率极低，合计不到 1%
   - 可以只关注一两种特定模式


## 7.2. Suggestions for Designing More Effective Strategies
1. 当前内联存在的三个主要问题
- 1. 现有方法默认两个函数都使用了内联，但实际上，只有**1%**的案例是这样的方式
  - 70% 不使用内联，29% 仅一端内联
- 2. 内联不必要的函数
- 3. 候选项减少
  -  现有方法通过限制内联深度或者过于严格的条件使得内联函数被遗漏
2. 建议
- 1. 预处理
  - 过滤用户指定不需要内联的函数，`__attribute__(noinline)`
  - 编译等级 `O0`
- 2. 放宽 `BINGO` 和 `ASM2VEC` 的条件
  - 过于苛刻的条件会导致遗漏很多内联函数
- 3. 

# 8. DISCUSSION
## 8.1. Study Settings
1. 直接分析编译器的设计相较于分析二进制文件更加困难
2. 内联的结果受到多方面因素影响
   - 内联策略
   - 项目差异
3. 使用内联策略作为补充信息，或许可以帮助检查二进制文件的内联
4. 内联除了会影响 BCSD 的检测，还会影响
   - 1. 函数调用图
   - 2. 指令排序和基本块组合
   - 3. 动态方法，但是影响较小


## 8.2. Effectiveness of Function Inlining Identification Method


# Related Knowledge
## 1. 函数内联
- 定义
  - 一种编译器优化技术，其主要目的是提高程序的执行效率
- 方法
  - **被调用函数的代码直接插入到调用函数的位置**，来减少函数调用的开销，从而加速程序运行


## 2. CFG
- 表示一个二进制函数中的基本块和它们之间的控制流关系，用于分析函数的**执行逻辑**


## 3. OSS
- Open Source Software 开源软件，源代码公开并允许任何人使用


## 4. 0-day 漏洞 与 1-day 漏洞
- 0-day
  - 未知或者未公开的漏洞
- 1-day
  - 已知漏洞，但是未安装补丁修复


## 5. `-g` 与 `.debug_line`
- `-g`
  - 编译器的一个选项，用于生成调试信息
  - 当使用-g选项编译代码时，编译器会在二进制文件中加入源代码的调试信息
    - 源代码行号、变量名、函数名称 etc
  - 以 **Dwarf** 格式输出
- `.debug_line`
  - Dwarf 格式
  - 记录了**二进制地址和源代码行号**的映射关系


## 6. 编译时优化（LTO）
- **链接阶段**对整个程序的**所有编译单元**进行全局分析和优化
  - 即可以**跨文件进行内联**
- 区别
  - 单个过程内联（没有 LTO）
    - 在没有启用LTO时，编译器只能对**同一个编译单元（即一个源文件或模块）**内的函数调用进行内联
    - 比如，`file1.c` 文件中定义了一个函数 `foo()`，并在**同一文件中**调用 `foo()`，编译器可以将 foo() 内联到调用位置。\
    - 但是，如果 `file2.c` 中调用了 `file1.c` 中的 `foo()`，因为它们属于**不同的编译单元**，编译器无法在链接之前看到跨文件的函数定义，所以**不能内联该调用**
  - 跨过程内联（启用 LTO）
    - 启用LTO后，编译器可以在**链接阶段**查看**整个程序**的所有**编译单元的代码**，因此**可以进行跨文件或跨模块**的内联。
    - 例如，`file2.c` 调用 `file1.c` 中的 `foo()` 时，因为LTO允许编译器在链接阶段分析整个程序的代码，编译器能够看到 `foo()` 的实现，因此可以将它内联到 `file2.c` 中的调用位置