Title: 02-cython的编译方式
Date: 2010-12-03 10:20
Modified: 2010-12-05 19:30
Category: Python
Tags: cython
Authors: sniper
Summary: cython的编译方式


- [编译过程](#编译过程)
- [环境搭建](#环境搭建)
- [一、distutils 手动编译](#一distutils-手动编译)
  - [1.1 只编译Cython代码](#11-只编译cython代码)
  - [1.2 编译引入C代码的Cython代码](#12-编译引入c代码的cython代码)
  - [总结](#总结)
- [二、通过 IPython 动态交互 Cython](#二通过-ipython-动态交互-cython)
- [三、导包的时候编译](#三导包的时候编译)
  - [3.1 使用 pximport 即时编译](#31-使用-pximport-即时编译)
  - [3.2 控制 pyximport 并管理依赖](#32-控制-pyximport-并管理依赖)


# 编译过程

1. cython 编译器负责将 Cython 转换成经过优化并且依赖当前平台的 C、C++ 代码

2. 使用标准的 C、C++ 编译器将第一步得到的 C、C++ 代码进行编译并生成标准的扩展模块

    - 扩展模块依赖特定的平台. linux平台, 编译生成so扩展; windows平台, 编译生成pyd扩展
    - 扩展模块可以直接被 Python 解释器进行 import


# 环境搭建

1. 安装C、C++编译器

    - linux平台
      - 自带gcc, 无需安装
      - yum install python3-devel

    - windows平台
      - 下载Visual Studio 或者 安装[MinGW](https://sourceforge.net/projects/mingw/files/)并设置到环境变量中
      - [详见](https://zhuanlan.zhihu.com/p/471661231)


2. 安装Cython编译器

    pip install cython


# 一、distutils 手动编译


## 1.1 只编译Cython代码
  [详见](https://gitlab.com/pymango/cython/-/tree/master/Cython%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%B2%BE%E9%80%9A/Cython%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91/build_cython)

1. 编写编译脚本build.py
    ```python
    from distutils.core import setup
    from Cython.Build import cythonize

    setup(ext_modules=cythonize("fib.pyx", language_level=3))
    ```

2. 执行编译命令

    `python.exe build.py build`

## 1.2 编译引入C代码的Cython代码
  [详见](https://gitlab.com/pymango/cython/-/tree/master/Cython%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%B2%BE%E9%80%9A/Cython%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91/build_cython_c)

1. 编写编译脚本build.py
    ```python
    from distutils.core import setup, Extension
    from Cython.Build import cythonize

    ext = Extension(name="fib", sources=["fib.pyx", "cfib.c"])
    setup(ext_modules=cythonize(ext))
    ```

2. 执行编译命令

    `python.exe build.py build`

3. 可能出错

    - [解决方案参考](https://blog.csdn.net/weixin_41423872/article/details/124407109)

## 总结

- 如果是单个 pyx 文件的话, 那么直接通过 cythonize("xxx.pyx") 即可
- 如果 pyx 文件还引入了 C 文件, 那么通过 cythonize(Extension(name="xx", sources=["", ""])) 的方式即可; name 是编译之后的扩展模块的名字, sources 是你要编译的源文件


# 二、通过 IPython 动态交互 Cython
- jupyter notebook同理

```python
# 我们在 IPython 上运行，执行 %load_ext cython 便会加载 Cython 的一些魔法函数
In [1]: %load_ext cython

# 然后神奇的一幕出现了，加上一个魔法命令，就可以直接写 Cython 代码
In [2]: %%cython
   ...: def fib(int n):
   ...:     """这是一个 Cython 函数，在 IPython 上编写"""
   ...:     cdef int i
   ...:     cdef double a = 0.0, b = 1.0
   ...:     for i in range(n):
   ...:         a, b = a + b, a
   ...:     return a
_cython_magic_0c221c85b2c3d523d965f7911ee5a741.c
_cython_magic_0c221c85b2c3d523d965f7911ee5a741.obj : warning LNK4197: export 'PyInit__cython_magic_0c221c85b2c3d523d965f7911ee5a741' specified multiple times; using first specification
   Creating library C:\Users\CP\.ipython\cython\Users\CP\.ipython\cython\_cython_magic_0c221c85b2c3d523d965f7911ee5a741.cp39-win_amd64.lib and object C:\Users\CP\.ipython\cython\Users\CP\.ipython\cython\_cython_magic_0c221c85b2c3d523d965f7911ee5a741.cp39-win_amd64.exp
Generating code
Finished generating code

# 测试用时, 平均花费34.2ns
In [3]: %timeit fib(50)
34.2 ns ± 0.399 ns per loop (mean ± std. dev. of 7 runs, 10,000,000 loops each)
```


# 三、导包的时候编译

## 3.1 使用 pximport 即时编译
[代码见](https://gitlab.com/pymango/cython/-/tree/master/Cython%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%B2%BE%E9%80%9A/Cython%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91/pxinport_compile)
- Cython 源文件不会立刻编译, 只有当被导入的时候才会编译
- Cython 源文件不变, 不会重复编译; Cython 源文件被修改, pyximport 会自动检测, 会重新编译

```python
# add.pyx
def add(int a, int b):
    return a + b

# build.py
import sys
sys.path.append('.')

import pyximport
# 这里同样指定 language_level=3, 则表示针对的是py3, 因为这种方式也是要编译的
pyximport.install(language_level=3)
# 执行完之后, Python 解释器在导包的时候就会识别 Cython 文件了, 当然会先进行编译

import add
print(add.add(11, 22))
```

## 3.2 控制 pyximport 并管理依赖
[代码见](https://gitlab.com/pymango/cython/-/tree/master/Cython%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E7%B2%BE%E9%80%9A/Cython%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91/pyximport_compile)
- .pyxbld 文件要和 .pyx 文件具有相同的基名, 且它们要位于同一目录中

```python
# fib.pyxbld
from distutils.extension import Extension

def make_ext(modname, pyxfilename):
    return Extension(modname,
                     sources=[pyxfilename, "../build_cython_c/cfib.c"],
                     include_dirs=["../build_cython_c/"])

# build.py
import sys
sys.path.append('.')

import pyximport
pyximport.install(language_level=3)

import fib
print(fib.fib_with_c(50))
```
