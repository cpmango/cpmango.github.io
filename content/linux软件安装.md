Title: linux软件安装
Date: 2010-12-03 10:20
Modified: 2010-12-05 19:30
Category: Linux
Tags: picgo,navicat,typora,flameshot
Slug: AppImage
Authors: sniper
Summary: linux软件安装


- [picgo](#安装picgo)
- [navicat15](#破解并安装navicat15)
- [typora](#安装typora)
- [flameshot](#安装火焰截图flameshot)

# AppImage格式

## 安装picgo

1. 下载PicGo-2.3.1.AppImage

​		[山东大学镜像站](https://mirrors.sdu.edu.cn/github-release/Molunerfinn_PicGo/v2.3.1/PicGo-2.3.1.AppImage)

​		[百度网盘](https://pan.baidu.com/s/10M3qut16jisEK4k69RsaPg?pwd=h3rs)

2. 创建一个用于存放软件的文件夹

   ```shell
   mkdir /mnt/linux/softwares/picgo-2.3.1
   mv PicGo-2.3.1.AppImage /mnt/linux/softwares/picgo-2.3.1/
   ```

3. 下载一张picgo的软件图标命名为picgo.png

   ```shell
   mv picgo.png /mnt/linux/softwares/picgo-2.3.1/
   ```

4. 编辑picgo.desktop

   ```shell
   cd ~/.local/share/applications
   vim picgo.desktop
   ```

5. picgo.desktop内容如下

   ```
   [Desktop Entry]
   Type=Application
   Name=picgo
   Exec=/mnt/linux/softwares/picgo-2.3.1/PicGo-2.3.1.AppImage
   Icon=/mnt/linux/softwares/picgo-2.3.1/picgo.png
   Categories=Graphics
   ```

   - Categories：代表软件类型，详细可参考： [Desktop文件Categories详细说明](https://blog.csdn.net/shawzg/article/details/106943100)


6. 创建软链接

   ```shell
   sudo ln -snf /mnt/linux/softwares/picgo-2.3.1/PicGo-2.3.1.AppImage /usr/local/bin/picgo
   ```

   - 为后面配置Typora图像上传而设置

## 破解并安装navicat15

1. 破解

   - 参照 [破解教程](https://github.com/pchen2022/navicat-keygen-tools/blob/main/README.zh-CN.md)

   - navicat15下载 [navicat15-en](https://pan.baidu.com/s/1bUY0QVKuzIwIjAjhg0vDFQ?pwd=i62x)
   - 破解工具下载
     - [patcher](https://pan.baidu.com/s/1STy-pO1iLmbGtZ4hajBUKQ?pwd=trb5)
     - [keygen](https://pan.baidu.com/s/1nDsDFTrkpsXpdp9JK7VCFA?pwd=djvw)
2. 啊



# deb格式

## 安装typora

1. 下载typora_0.11.18_amd64.deb
   - 0.11.18版本是官网提供的免费版
   - 之后的版本都需要破解

​		[百度网盘](https://pan.baidu.com/s/1W21O5Xqy2fq6s6_wh-_Jfg?pwd=u6mj)

2. 双击即可安装



# apt命令安装

## 安装火焰截图flameshot

1. 安装

   ```shell
   sudo apt install flameshot
   ```

2. 配置

   系统设置 -> 键盘 -> 快捷键 -> 添加自定义快捷键

   ![image-20230113231051827](https://i.328888.xyz/2023/01/13/w7pm3.png "211848495607274-pchen-20230113231051-f90141d5737f1179fd816028b28c5df3.png")

   ![image-20230113231146313](https://i.328888.xyz/2023/01/13/w7Bvy.png "657964904373742-pchen-20230113231146-3c121f51c209f2a7e91bba4d1859cef5.png")

   ![image-20230113231207352](https://i.328888.xyz/2023/01/13/w7gy5.png "920241347431163-pchen-20230113231207-47a1e7ae248e94d5383b46cd58d50422.png")