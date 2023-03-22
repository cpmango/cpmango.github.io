Title: 03-使用Cython释放GIL实现并行执行
Date: 2010-12-03 10:20
Modified: 2010-12-05 19:30
Category: Python
Tags: cython
Authors: sniper
Summary: 使用Cython释放GIL实现并行执行


- [一、全局解释器锁(GIL)](#一全局解释器锁gil)
  - [1.1 GIL是什么](#11-gil是什么)
  - [1.2 为什么会有 GIL](#12-为什么会有-gil)
  - [1.3 python线程机制](#13-python线程机制)
  - [1.4 GIL调度机制](#14-gil调度机制)
- [二、Cython绕过GIL限制](#二cython绕过gil限制)
  - [2.1 为什么Cython可以绕过GIL](#21-为什么cython可以绕过gil)
  - [2.2 拓展python动态性](#22-拓展python动态性)
  - [2.3 Cython 中如何管理 GIL](#23-cython-中如何管理-gil)
    - [2.3.1 nogil 函数属性](#231-nogil-函数属性)
    - [2.3.2 with nogil 上下文管理器](#232-with-nogil-上下文管理器)


# 一、全局解释器锁(GIL)

## 1.1 GIL是什么

- GIL 是一个 施加在<font color='green'>*解释器*</font>层面上, <font color='green'>*字节码*</font>级别的互斥锁
- 用于防止本机多个线程同时执行字节码, 保证每一条字节码在执行的时候都不会被打断
- GIL 确保 CPython 在程序执行期间，同一时刻只会使用操作系统的一个线程。不管你的 CPU 是多少核，以及你开了多少个线程，但是同一时刻只会使用操作系统的一个线程、去调度一个 CPU。而且 GIL 不仅影响 Python 代码，也会影响 Python/C API
- GIL 不是 Python 语言的特性, 是 <font color='green'>**CPython解释器**</font>开发人员为了方便内存管理加上去的

## 1.2 为什么会有 GIL

```python
import dis

dis.dis("del name")
"""
  1           0 DELETE_NAME              0 (name)
              2 LOAD_CONST               0 (None)
              4 RETURN_VALUE
"""
```

- python 使用 del 删除一个变量的时候, 对应的指令是 DELETE_NAME
- 这条指令做的事情就是通过宏 Py_DECREF 减少一个对象的引用计数，并且判断减少之后其引用计数是否为 0，如果为 0 就进行回收. 概括为2步
    1. 将对象的引用计数减 1
    2. 判断引用计数是否为 0，为 0 则进行销毁

```c
--obj->ob_refcnt
if (obj -> ob_refcnt == 0){
	销毁obj
}
```

- 引入的问题：假设有两个线程 A 和 B，内部都引用了全局变量 obj，此时 obj 指向的对象的引用计数为 2，然后让两个线程都执行 del obj 这行代码

    - 其中 A 线程先执行，如果 A 线程在执行完 --obj->ob_refcnt 之后，会将对象的引用计数减一，但不幸的是这个时候调度机制将 A 挂起了，唤醒了 B。而 B 也执行 del obj，但是它比较幸运，将两步都一块执行完了。但由于之前 A 已经将引用计数减1，所以 B 线程再减 1 之后会发现对象的引用计数为0，从而执行了对象的销毁动作，内存被释放

    - 然后 A 又被唤醒了，此时开始执行第二个步骤，但由于 obj->ob_refcnt 已经被减少到 0，所以条件满足，那么 A 依旧会对 obj 指向的对象进行释放，但是这个对象所占内存已经被释放了，所以 obj 此时就成了悬空指针。如果再对 obj 指向的对象进行释放，最终会引发什么结果

- 解决问题: 引入GIL

  - 由于引入 GIL，所以就不存在：在 A 将引用计数减一之后，挂起 A、唤醒 B 这一过程。因为A已经开始了 DELETE_NAME 这条指令的执行，所以在没执行完之前是不会发生线程调度的，此时就不会发生悬空指针的问题

## 1.3 python线程机制

![image-20230115095419874](https://i.328888.xyz/2023/01/15/2nR1C.png "178901278710458-pchen-20230115095420-bbc5544b69f15182c4c72ba903ab4caf.png")

- Python 线程是调用 C 的线程、进而调用操作系统的 OS 线程
- 每个线程在执行过程中 Python 解释器控制不了，因为 Python 的控制范围只有在解释器这一层，无权干预 C 的线程、更无权干预 OS 线程

![image-20230115100232741](https://i.328888.xyz/2023/01/15/2ngDJ.png "524915507745237-pchen-20230115100232-dc429b7ed68de4d8f7fed9bed09c29a8.png")

## 1.4 GIL调度机制

1. 当遇见 io 阻塞的时候会把锁释放, io 阻塞是不耗费 CPU 的, 此时虚拟机会把该线程的锁释放
2. 为了保证每个线程都有机会执行, 即便是耗费 CPU 的运算, 也会在执行一小段时间之后释放锁

```shell
import sys
print(sys.getcheckinterval())  # 100
sys.setcheckinterval(N)  # 手动设置
```

python3.8之前, 默认是执行 100 条字节码启动线程调度机制，进行切换

```shell
import sys
print(sys.getswitchinterval())  # 0.005
sys.setswitchinterval(N)  # 手动设置
```

现在, 更应该使用这个函数，表示线程切换的时间间隔


# 二、Cython绕过GIL限制

## 2.1 为什么Cython可以绕过GIL

- GIL是在解释器解释执行字节码的时候所施加的
- Cython 代码经过编译之后直接指向了 C 一级的结构，所以它相当于绕过了解释器解释执行这一步
- 所以当我们在 Cython 中创建了不绑定任何 Python 对象的 C 级结构时, 可以将GIL释放掉

## 2.2 拓展python动态性

-  Python 的动态性，也是在解释器解释字节码的时候所赐予的
-  Cython直接编译成C代码, 失去了相应动态特性
-  所以能带来速度的提升

## 2.3 Cython 中如何管理 GIL

### 2.3.1 nogil 函数属性

### 2.3.2 with nogil 上下文管理器