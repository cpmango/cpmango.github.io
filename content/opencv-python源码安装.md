Title: opencv-python源码安装
Date: 2010-12-03 10:20
Modified: 2010-12-05 19:30
Category: python
Tags: opencv-python
Slug: cv
Authors: sniper
Summary: opencv-python源码安装


- [背景介绍](#背景介绍)
- [设置环境变量](#设置环境变量)
- [一、安装x264](#一安装x264)
- [二、安装ffmpeg](#二安装ffmpeg)
- [三、编译opencv-python](#三编译opencv-python)
- [补充:静态库和动态库的区别](#补充静态库和动态库的区别)
- [引用](#引用)

# 源码编译支持h264编码的opencv python库

## 背景介绍

1. FFmpeg默认支持H264的解码，但是并不支持H264的编码，如果想要让FFmpeg支持H264编码就要从外部引入X264进FFmpeg
2. <font color='red'> FFmpeg对第三方库通常是优先使用动态链接的 (为什么？怎么判断出来的？) </font>


## 设置环境变量

```shell
mkdir /home/sniper/local
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/home/sniper/local/bin
export LD_LIBRARY_PATH=/home/sniper/local/lib
export PKG_CONFIG_PATH=/home/sniper/local/lib/pkgconfig
```


## 一、安装x264

```shell
# 下载源码
git clone https://github.com/mirror/x264.git
# 编译
cd x264/
./configure --enable-shared  --disable-asm  --prefix=/home/sniper/local
```

- --enable-shared 表示生成动态库，如生成静态库将--enable-shared替换为--enable-static
- --prefix 生成库安装路径

```shell
# 安装
make && make install
# 查看版本信息
x264 --version
```


## 二、安装ffmpeg

```shell
# 下载源码
git clone https://github.com/FFmpeg/FFmpeg.git
# 编译
cd FFmpeg/ && mkdir build && cd build
../configure --enable-shared --enable-libx264 --enable-gpl --disable-x86asm --prefix=/home/sniper/local
# 安装
make && make install
# 查看是否支持h264编码
ffmpeg -encoders | grep h264
```


## 三、编译opencv-python

```shell
# 下载源码
git clone -b 64 https://github.com/opencv/opencv-python.git
# 添加版本信息
echo -e "opencv_version = '4.5.5.64'\ncontrib = False\nheadless = False\nci_build = False" > cv2/version.py
# 修改setup.py
sed -i "s/cmake_install_dir=cmake_install_reldir/_cmake_install_dir=cmake_install_reldir/g" setup.py
# 下载opencv源码
rm -rf opencv && git clone -b 4.5.5 https://github.com/opencv/opencv.git
# 安装、更新依赖
/home/sniper/py38/bin/pip install scikit-build numpy
/home/sniper/py38/bin/pip install --upgrade pip setuptools wheel
# 编译
rm -rf .git && /home/sniper/py38/bin/python setup.py bdist_wheel --build-type=Debug
```

编译完成后, 会在dist目录下生成whl文件, 通过pip install dist/*.whl即可安装

```shell
# 验证
/home/sniper/py38/bin/python -c "import cv2"
```

其中还可能遇到world库相关报错时, 需要重新链接world相关的动态库


## 补充:静态库和动态库的区别

1. 库
   - 库是写好的，现有的，成熟的，可以复用的代码
   - 库是一种可执行代码的二进制形式，可以被操作系统载入内存执行
   - 库有两种：静态库（.a、.lib）和动态库（.so、.dll）
2. 所谓静态、动态是指链接, 即静态库、动态库区别来自**链接阶段**如何处理
3. [详见-静态库和动态库的区别](https://www.cnblogs.com/codingmengmeng/p/6046481.html)


## 引用

1. [linux下编译ffmpeg 引入外部库x264](https://www.cnblogs.com/wanggang123/p/8660435.html)
2. [ERROR: x265 not found using pkg-config的解决方法](https://blog.csdn.net/mls805379973/article/details/103425343)
3. [解决ippicv下载慢的问题](https://blog.51cto.com/SpaceVision/3086952)
4. [OpenCV中手动安装ippicv](https://www.jianshu.com/p/3c2fc0da7398)
5. [在 Linux 系统中编译安装 OpenCV](https://zhuanlan.zhihu.com/p/392751819)