# 1. Problem
## 1.1. 函数调用数量发生显著改变
### 1.1. 2022_B_Practical Binary Code Similarity Detection with BERT-based Transferable Similarity Learning
```cpp
// source: 
//      binary_file: blender_r from SPEC2017 suite
static void emDM_drawEdges(DerivedMesh *dm, int UNUSED(drawLooseEdges), int UNUSED(drawAllEdges))
{
    emDM_drawMappedEdges(dm, NULL , NULL);
}
```
1. 在 `O0` 下（不内联），`emDm-drawEdges` 的调用函数数量为 `1`，即 `emDm_drawMappedEdges`
2. 在 `O3` 下（发生内联），`emDm-drawEdges` 的调用函数数量为 `7`，即 `emDm_drawMappedEdges` 的调用函数被作为 `emDm-drawEdges` 的调用函数



## 1.2. 指令数量发生改变
### 1.2.1. 2022_B_Practical Binary Code Similarity Detection with BERT-based Transferable Similarity Learning
1. 
```cpp
// source: 
//      binary_file: blender_r from SPEC2017 suite
static void emDM_drawEdges(DerivedMesh *dm, int UNUSED(drawLooseEdges), int UNUSED(drawAllEdges))
{
    emDM_drawMappedEdges(dm, NULL , NULL);
}
```
1. 在 `O0` 下（不内联），`emDm-drawEdges` 的指令数量为 13
2. 在 `O3` 下（发生内联），`emDm-drawEdges` 的指令数量为 56



# 2. Method
