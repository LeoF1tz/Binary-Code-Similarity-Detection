# 1. 2022_B_Practical Binary Code Similarity Detection with BERT-based Transferable Similarity Learning
1. 问题定义：假设每个函数中只含有一个样本，即符合 `FewShot` 的特定情况，即 `N way 1 shot`
2. 损失函数：二元交叉熵
   - $L=Y\ log\ p(X_i,X_j)+(1-Y)\ log\ (1-p(X_i,X_j))$


# 2. 