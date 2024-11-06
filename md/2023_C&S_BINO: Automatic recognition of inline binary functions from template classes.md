| **Title** | BINO: Automatic recognition of inline binary functions from template classes |
|   -|    -|
| **Author** | Lorenzo Binosi |
| **Institution** | Politecnico di Milano |
| **Venu** | C&S |
| **Year** | 2023 |


# ABSTRACT
1. 本文方法
   1. 使用**二进制指纹识别和匹配**的方式识别内联调用
      1. 为每个内联函数创建了一个指纹，该指纹包含了函**数的语法、语义特征和 CFG**
      2. 使用**子图同构算法**匹配并识别嵌入在其他代码中的内联函数
   2. 实现自动化指纹生成用于匹配
2. 实验结果
   - `BINO` 在识别内联函数方面的 F1 可达到 63%

 

# CONCLUSION
1. 本文方法
   1. 基于指纹的内联函数识别方法
   2. 实现自动化生成和匹配指纹
2. 实验结果
   1. `BINO` 在使用 `-O2`、`-O3` 和 `-Ofast` 优化级别时，能够以** 63% 的 F1 分数**成功识别出一些常用模板类方法的内联调用
   2. 在更为复杂的优化等级下，效果不理想（ `-Os`
3. 目前局限性
   1. 仅支持**基本块级别**
   2. 无法识别内联函数在基本块的**具体位置**

 

# 1. INTRODUCTION
1. 识别内联函数的难点
   - CFG 中包含多个子图，即包含多个函数
2. 指纹模型的作用
   - 捕捉汇编函数的语法、语义特征以及控制流程图结构
3. 本文主要贡献
   1. `BINO` 是第一个针对识别模板内联的方法
   2. 提出一个指纹模型，用于捕捉函数特征
   3. 开发了一个二进制指纹和匹配框架，能够从模板类的源代码生成相关的指纹
   4. `BINO` 性能评估

 

# 2. Background and motivation
## 2.1. Inlining
1. 函数内联的定义：在编译时将函数调用替换为该函数的实际代码
2. 函数内联的优点：优点避免调用指令和减少函数额外代码的影响
3. 函数内联的缺点：重复的函数代码会占用更多的指令缓存，增加二进制文件的大小

 

## 2.2. Problem statement
1. 内联库函数在 C++ 中被广泛使用
2. 模板会经常使用到多级内联
3. i.e.,
``` C++
void add_string_to_vector(std::vector<std::string>& vec, const std::string& std){
    // 将 std 加入到数组 vec 中
    vec.push_back(std);
}
```
- `add_string_to_vector` 涉及的内联
  1. `push_back` >> `str::stding` 
  2. `std::stding` 
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img.png>)
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-1.png>)
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-2.png>)
4. 图解
   1. (a)：`O0` 优化等级，即无优化，编译器保留每一个函数调用
   2. (b)：`O3` 优化等级，编译器将 `std::vector::push_back` 和 `std::vector` 的内部实现 `std::vector::_M_realloc_insert` 内联进，即发生了**多级内联**

 

### 2.2.1. Multilevel Inlining
1. 多级内联的定义
   - **递归地内联函数**
2. i.e.,
   `foo` >> `bar` >> `baz` 

 

### 2.2.2. Template Parameters
1. 模版类会根据**不同的数据类型**生成不同的代码
   - `std::vector<int>` 和 `std::vector<std::string>`
2. 模版类的汇编代码可能设计多级内联
   - `std::vector<std::string>` >> `push_back` >> `std::string`

 

### 2.2.3. Constant Folding - Constant propagation - Dead Code Elimination
1. 函数内联的影响
   - 将函数的部分内容替换为被调用函数的主体后，调用方可以**直接访问**被调用函数的代码和变量
2. 种类
   1. 常量折叠：将编译时已知的常量表达式直接计算出结果
      - `3 + 5` >> `8`
   2. 常量传播：已知的常量值传播到程序的各个部分
      - 果调用方在传递给内联函数的参数是一个已知的常量，编译器可以在内联函数中用**该常量直接替代参数变量**
   3. 死代码消除：移除永远不会执行的代码
      - 如果编译器在内联过程中知道一个 `if` 语句的结果，它可以直接移除无用的分支代码

 

### 2.2.4. Initial and Final Instructions
- 基本块中存在内联函数和被内联函数的**混合代码**

 

# 3. IDENTIFICATION OF INLINING FUCNTIONS
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-3.png>)


## 3.1. Fingerprint generation（todo）
1. 助记符“颜色”分组
   - ![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-13.png>)
2. 函数调用信息
   - ![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-14.png>)
3. 指纹生成
   - CFG + 助记符组 + 函数调用信息 = 指纹 
 

### 3.1.1. Parser
- 使用 `clang-cindex` 构建源代码的 AST，提取类的公共方法信息
  - **方法的名称、参数的数据类型和返回值的数据类型**


### 3.1.2. Code Generator
1. 目的
   1. 选择适合进行编译和指纹提取的代码样本
2. 方法
   1. 标记起始和结束：插入 `asm` 指令，明确内联方法的开始和结束位置
   2. 汇编代码保留**内联方法的所有部分**
      1. 在包装函数外部声明对象：内联函数设计的对象声明在包装函数的外部
      2. 确保内联函方法的返回值被使用
      3. 使用标记指令 `asm`
   3. 为模版中每种不同类型的参数生成相应代码
      - i.e., 对 `std::vector` 中的所有数据类型生成不同的代码文件，e.g., `int`, `double`, etc.


### 3.1.3. Compiler
- 每个代码在不同优化等级下编译，其使用了不同程度的内联
  - e.g., `O2`, `O3`, `Os`, `


### 3.1.4. Fingerprinter
1. 目的
   - 从二进制文件中提取指纹
2. 指纹提取方法
   1. 使用 `Angr` 进行反编译，提取
      1. 汇编指令
      2. 函数调用信息
      3. CFG：`Angr` 将函数的 CFG 按基本块进行分割
   2. 移除无关的汇编指令：只包含内联函数的代码
   3. 构建指纹，其特征包含**颜色**和***函数调用信息**
      1. 颜色：位向量形式，表示指令类型
      2. 函数调用信息：调用的函数名称和类型


### 3.1.5. Reducer
1. 指纹重复问题
   - 源代码可能经过不用的编译选项编译后生成**相同指纹**
2. 寻找重复指纹的方法
   1. 使用 VF2 算法查找具有相同 CFG 的指纹
      - 两者结构上一一对应
   2. 检查对应基本块的颜色位向量和函数调用信息
   3. 移除重复指纹
      - 如果在至少一个映射中，所有基本块的颜色和调用信息完全相同，则可以确定这两个指纹是完全重复的


## 3.2. Fingerprint matching
1. 生成目标指纹：将目标函数反编译，生成 CFG
2. 子图同构匹配：查找与目标函数的 CFG 的**子图**具有相同结构的查询函数
   - 目标函数是子图，查询函数是整个图
   - 设置一个**基本块阈值**，高于该阈值才会被作为查询函数
   - 查询函数的 CFG 生成三种变体以应对函数内联导致的 CFG 变化
3. 指纹特征比较：实现匹配后，对函数调用信息和颜色信息进行匹配
4. 输出：输出识别出的内联方法列表以及这些方法在二进制文件中对应的基本块的地址


### Appendix CFG Variations
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-4.png>)
面对内联使得 CFG 发生变化，`BINO` 会对**查询函数**进行三种自动变体
2. 循环结构
3. 无条件跳转结构
4. 循环和无条件跳转结合，即组合结构


# 4. EVALUATION
## 4.5. `BINO` Matching performance
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-5.png>)
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-6.png>)
1. 在 `O2`, `O3`, `Ofast` 优化等级下，达到 72% 的精度和 56% 的召回率
2. 在 `Os` 这种针对空间优化下，精度略显下降


## 4.6. Model transferability
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-7.png>)
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-8.png>)
- `BINO` 有一定的泛化能力，即可部分识别未见过的种类
- ~~对已知种类的识别有些结果并不理想~~

## 4.7. Fine parameters selection
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-9.png>)
- 对特定任务选择特定的超参数可显著提高模型性能


## 4.8. Fingeprint complexity analysis
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-10.png>)



## 4.9. Impact of syntactic and semantic features
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-11.png>)
![alt text](<images/2023_C&S_BINO: Automatic recognition of inline binary functions from template classes/img-12.png>)
1. 从 `F1-Score` 分析，函数调用信息和颜色特征相较于只依靠 CFG，性能显著提升
2. 影响力大小：颜色 >> 函数调用信息
   - 颜色特征在每个基本块中都存在，但是函数调用信息只存在于调用其他函数的基本块中
3. 函数调用信息和颜色显著提升了在**基本块数量较少**情况下的准确率，如果基本块数量足够多，CFG 就可以提取足够多的特征


# 5. LIMITATIONS AND FUTURE WORKS
1. 过小的函数缺乏特征，无法准确识别，假阳性率过高
   - 可以提取对象的内存地址和寄存器引用
   - 追踪对象的生命周期
2. 编译器如果过度优化，导致 CFG 发生显著改变，进而导致子图难以匹配，影响准确率
3. `BINO` 只能识别基于基本块的边界
4. `BINO` 可拓展为跨架构


# Related Knowledge
1. 优化等级
- 由小到大：`-O0`, `-O1`, `-O2`, `-O3`, `-Ofast`
- 针对代码大小：`-Os`，其与 `-O2` 类似，但是特别关注**代码体积**


2. 指纹匹配
- 定义：通过创建和比较函数、模块或库的特征“指纹”来识别它们