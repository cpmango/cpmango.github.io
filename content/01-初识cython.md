Title: 01-初识cython
Date: 2010-12-03 10:20
Modified: 2010-12-05 19:30
Category: Python
Tags: cython
Authors: sniper
Summary: 初识cython


- [Cython使用场景](#cython使用场景)
- [Cython是什么](#cython是什么)
- [Cython 和 CPython 的区别](#cython-和-cpython-的区别)
- [Python、C、C扩展和Cython效率对比](#pythoncc扩展和cython效率对比)
  - [斐波那契数列代码](#斐波那契数列代码)
  - [测试结果](#测试结果)
  - [差异分析](#差异分析)
  - [总结](#总结)
  

# Cython使用场景

1. 因为某些需求导致不得不编写一些多重嵌套的循环, 而这些循环如果用 C 语言来实现会快几百倍, 但是不熟悉 C 或者不知道 Python 如何与 C 进行交互.

2. 因为 Python 解释器的性能原因, 如果将 CPython 解释器换成 PyPy, 或者干脆换一门语言, 比如 Julia, 将会得到明显的性能提升, 可是换不得.因为你的项目组规定只能使用 Python 语言, 解释器只能 CPython.
   
3. Python 是一门动态语言, 但你希望至少在数字计算方面, 能够加入可选的静态类型, 这样可以极大的加速运算效果.因为单纯的数字相加不太需要所谓的动态性, 尤其是当你的程序中出现了大量的计算逻辑时.

4. 对于一些计算密集型的部分, 你希望能够写出一些超越 Numpy、Scipy、Pandas 的算法.

5. 你有一些已经用 C、C++ 实现的库, 你想直接在 Python 内部更好地调用它们, 并且不使用 ctypes、cffi 等模块.

6. 也许你听说过 Python 和 C 可以无缝结合, 通过 C 来为 Python 编写扩展模块, 将 Python 代码中性能关键的部分使用 C 进行重写, 来达到提升性能的效果.但是这需要你对 Python 解释器有很深的了解, 熟悉底层的 Python/C API, 而这是一件非常痛苦的事情.


# Cython是什么

1. Cython 是一门编程语言, 它将 C、C++ 的静态类型系统融合在了 Python 身上.
   - 文件的后缀是 .pyx, 是 Python 的一个超集；语法是 Python 语法和 C 语法的混血, 当然我们说它是 Python 的一个超集, 因此你写纯 Python 代码也是可以的.

2. cython 是一个编译器, 负责将 Cython 源代码翻译成高效的 C 或者 C++ 源代码；Cython 源文件被编译之后的最终形式可以是 Python 的扩展模块(.pyd), 也可以是一个独立的可执行文件.


# Cython 和 CPython 的区别

1. CPython 是 Python 语言对应的解释器

    - 解释型语言, 需要对应的解释器; 编译型语言, 需要对应的编译器
    - Python 语言解释器还有Jython(java实现)、PyPy(Python 语言实现)等等
    - CPython 是由 C 实现, 给 Python 语言提供 C 级别的接口, 也就是熟知的 Python/C API
      - Python 中的列表, 底层对应的是 PyListObject
      - Python 中的字典, 则对应 PyDictObject

2.  Cython 是一门语言, 可以通过 Cython 源代码生成高效的扩展模块, 同样需要 CPython 来进行调用.


# Python、C、C扩展和Cython效率对比

## 斐波那契数列代码

1. 纯Python
    ```python
    def fib(n):
        a, b = 0.0, 1.0
        for i in range(n):
            a, b = a + b, a
        return a
    ```

2. 纯C
    ```c
    double cfib(int n) {
        int i;
        double a=0.0, b=1.0, tmp;
        for (i=0; i<n; ++i) {
            tmp = a; a = a + b; b = tmp;
        }
        return a;
    }
    ```

3. C扩展

    ```c
    static PyObject *
    fib(PyObject *self, PyObject *n) {
        if (!PyLong_CheckExact(n)) {
            PyErr_Format(PyExc_ValueError, "function fib excepted int, not %s", Py_TYPE(n) -> tp_name);
            return NULL;
        }
        PyObject *z;
        double a = 0.0, b = 1.0, tmp;
        int i;
        for (i = 0; i < PyLong_AsLong(n); i++){
            tmp = a; a = a + b; b = tmp;
        }
        z = PyFloat_FromDouble(a);
        return z;
    }
    ```

4. Cython

    ```python
    def fib(int n):
        cdef int i
        cdef double a = 0.0, b = 1.0
        for i in range(n):
            a, b = a + b, a
        return a
    ```

## 测试结果

|          | fib(0)耗时/ns | 提升倍数 | fib(90)耗时/ns | 提升倍数 | 循环体耗时/ns | 提升倍数 |
| -------: | ------------: | -------: | -------------: | -------: | ------------: | -------: |
| 纯Python |           590 |        1 |          12852 |        1 |         12262 |        1 |
|      纯C |             2 |      295 |            164 |       78 |           162 |       76 |
|    C扩展 |           220 |        3 |            386 |       33 |           166 |       74 |
|   Cython |            90 |        7 |            258 |       50 |           168 |       73 |

- fib(0), 显然它没有真正进行循环, 测量的是调用一个函数所需要花费的开销
- 循环体耗时, 是执行 fib(90) 的时候, 排除函数调用本身的开销, 也就是执行内部循环体所花费的时间

## 差异分析

1. Python 调用一个函数的时候需要创建一个栈帧, 而这个栈帧是分配在堆上的, 而且结束之后还要涉及栈帧的销毁
2. C 扩展本质也是 C 语言, 只不过在编写的时候遵循 Python 提供的 API 规范, 可以将 C 代码编译成 pyd 文件, 直接让 Python 来调用
3. Cython 做的事情和 C 扩展本质是类似, 都是为 Python 提供扩展模块

---
- 算数操作方面对比
  - Python执行 a+b 等价于: 检测数据类型 -> 判断是否有__add__以及能否相加 -> 调用__add__方法, 将a 和 b 指向的对象进行相加 -> 指针转化成 PyObject * 返回
  -  C 和 Cython 创建变量的时候就实现规定了类型, 因此编译之后的 a + b 只是一条简单的机器指令
- 内存分配方面对比
  - Python 中的对象是分配在堆上面, 本质上就是 C 中的 malloc 函数为结构体在堆区申请的一块内存. 堆区进行内存的分配和释放需要付出很大的代价
  - Cython 分配的变量是分配在栈上, 由操作系统维护的，会自动回收，效率极高

## 总结

    并非所有的 Python 代码在使用 Cython 时, 都能获得巨大的性能改进. 比如: 内存密集、I/O 密集、网络密集