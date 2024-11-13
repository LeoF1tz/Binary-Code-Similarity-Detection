# 1. Problem
## 1.1. Function Mangling
### 1.1.1. 2022_B_Practical Binary Code Similarity Detection with BERT-based Transferable Similarity Learning
- `ld.gold`（来自 `binutils`） 和 `parest_r`（来自 `SPEC2017`）都引用了 `std` 库中的 `_M_get_Tp_allocator`，但是由于**命名空间的不同**，其被判定为不相似
- 它们使用了命名重整后，在对应的二进制文件中生成了不同的标识

