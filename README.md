# Mat+

`Mat+` 是一个轻量级 C++ 线性代数小项目，核心目标是用较少的代码实现一个可直接使用的矩阵类，并在此基础上提供常见的矩阵运算、线性方程组求解、行列式和逆矩阵计算。

从当前代码来看，这个项目更接近一个教学型/练手型矩阵库，而不是完整工业级数值计算库。它的优点是结构直接、功能集中、容易阅读；它的局限也很明显，例如缺少维度检查、边界检查和更成熟的数值稳定性设计。

## 项目结构

仓库里的代码基本可以分成 3 层：

### 1. 核心矩阵类型

- `Mat.hpp`
- `Mat.cpp`

这里定义了 `Mat` 类和最基础的矩阵能力：

- 动态分配矩阵内存
- 拷贝构造和赋值
- 元素访问：`operator()(row, column)`、`operator[]`
- 行列交换：`swap_row`、`swap_column`
- 基础运算：`+`、`-`、`*`
- 辅助构造：`zeros`、`zeros_like`、`eye`
- 转置：`transpose`

`Mat` 使用手写的 `double*` 内存管理，说明这个项目是从数据结构和运算符重载的基本训练出发搭起来的。

### 2. 高阶线性代数工具

- `utils.hpp`
- `utils.cpp`

这一层实现了更高阶的算法：

- `LUPQ(Mat A)`：带行列置换的分解
- `solve(Mat A, Mat y)`：通过 LUPQ 分解求线性方程组
- `det(Mat A)`：通过分解计算行列式
- `QR(Mat A)`：QR 分解
- `QR_solve(Mat A, Mat y)`：通过 QR 分解求解
- `inv(Mat A)`：逐列求解得到逆矩阵

可以把 `utils` 看成这个项目真正的“线性代数功能层”。

### 3. 扩展构造与拼接操作

- `extension.hpp`
- `extension.cpp`

这一层提供更偏“构造工具”的接口：

- `operator^(const Mat&, int)`：矩阵整数次幂
- `apply_func(...)`：对矩阵逐元素应用函数
- `row_vector(...)` / `column_vector(...)`
- `diag(...)`
- `row_cat(...)` / `column_cat(...)`
- `block(...)`：块矩阵拼接

## 头文件组织

推荐外部使用统一入口头文件：

```cpp
#include "include/MatPlus.hpp"
```

其中：

- `include/MatPlus.hpp` 会统一包含 `Mat.hpp`、`utils.hpp`、`extension.hpp`
- `include/` 目录更像是对外暴露的头文件目录
- 根目录下也保留了一套同名头/源文件，主要用于项目自身编译

## 现有源码状态分析

当前仓库能看出几个值得注意的点：

1. `Mat.hpp` 与 `include/Mat.hpp` 并不完全一致  
   `include/Mat.hpp` 的接口更完整，例如补上了 `const double &operator()(...) const`、更多运算符声明以及更适合输出的 `operator<<`。

2. `MatPlus.cpp` 是一份历史性的“聚合实现”  
   它把 `Mat.cpp` 直接 `#include` 进来，再重复实现了一部分高阶算法。按照当前仓库里的编译说明，正常构建静态库时使用的是 `Mat.cpp + utils.cpp + extension.cpp`，而不是 `MatPlus.cpp`。

3. `test.cpp` 是示例性质代码  
   它演示了逆矩阵、行列式、幂运算、矩阵拼接和块矩阵，但它的首行包含路径写成了：

```cpp
#include "include./MatPlus.hpp"
```

更合理的写法应当是：

```cpp
#include "include/MatPlus.hpp"
```

## 已实现功能

目前这个项目已经具备以下功能：

- 基础矩阵创建与访问
- 矩阵加减乘
- 标量与矩阵混合运算
- 单位矩阵、零矩阵生成
- 转置
- 行列交换
- 线性方程组求解
- 行列式计算
- 逆矩阵计算
- QR 分解
- 矩阵幂
- 向量、对角矩阵、块矩阵拼接

## 算法思路

### LUPQ 分解

`solve` 和 `det` 的核心依赖 `LUPQ`。实现思路是：

- 在剩余子矩阵中寻找主元位置
- 通过行交换和列交换调整矩阵
- 逐步消元得到下三角矩阵 `L` 和上三角矩阵 `U`
- 用置换矩阵 `P`、`Q` 记录交换过程

这类实现适合教学理解“矩阵分解如何支持求解和行列式计算”。

### QR 分解

`QR` 使用的是较直接的正交化过程。代码可读性不错，但作为数值算法实现，它更偏基础版，适合学习，不适合拿来和成熟数值库比较稳定性。

## 构建方式

仓库里已经包含一份静态库：

- 根目录：`libMatPlus.a`
- `build/` 目录：已有 `.o` 和静态库产物

如果想重新生成静态库，仓库里的 `Compilation Instruction.txt` 给出的思路是：

```bash
mkdir build
cd build
g++ -std=c++11 -c ..\Mat.cpp ..\utils.cpp ..\extension.cpp
ar rcs libMatPlus.a *.o
```

如果你的程序位于项目根目录，可以像下面这样链接已有静态库：

```bash
g++ -std=c++11 your_file.cpp -I. -L. -lMatPlus -o your_program
```

如果你链接的是 `build/` 目录下的库，则把 `-L.` 改成 `-Lbuild`。

## 最小使用示例

```cpp
#include "include/MatPlus.hpp"
#include <iostream>

int main() {
    double arr[9] = {1, 2, 3, 4, 1, 2, 1, 3, 3};
    Mat A(3, 3, arr);

    std::cout << "A =" << std::endl;
    std::cout << A << std::endl;

    std::cout << "det(A) = " << det(A) << std::endl;

    Mat B = inv(A);
    std::cout << "inv(A) =" << std::endl;
    std::cout << B << std::endl;

    return 0;
}
```

## 适合继续改进的方向

如果后续继续维护，这几个方向最值得优先处理：

- 为矩阵运算增加维度合法性检查
- 为元素访问增加边界检查
- 补齐 const-correctness 和接口统一性
- 清理根目录与 `include/` 目录中重复但不完全一致的声明
- 明确 `MatPlus.cpp` 是否保留
- 补充更系统的测试用例
- 优化 QR / 分解算法的数值稳定性
- 增加 CMake 或更明确的跨平台构建脚本
