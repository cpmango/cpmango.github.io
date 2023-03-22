Title: typora+picgo搭建博客
Date: 2010-12-03 10:20
Modified: 2010-12-05 19:30
Category: tools
Tags: typora,picgo
Slug: tool
Authors: sniper
Summary: typora+picgo搭建博客


# Typora+PicGo+Gitee搭建博客写作环境

## 一、安装typora

[详见](https://phen.coding.net/p/book-wiki/d/init-linux-env/git/tree/master/install_software.md)

## 二、创建Gitee图床仓

1. 新建名为oss的仓库，初始化仓库并设置为开源，并创建一个images的目录

   ![image-20230113224825150](https://i.328888.xyz/2023/01/13/wxEgd.png "920222550057233-pchen-20230113224825-6e8a3558ca0d2d18c0e1ffcd827a0b09.png")

2. 设置私人令牌

   ![image-20230113225247458](https://i.328888.xyz/2023/01/13/wx5jx.png "970169486057026-pchen-20230113225247-fb7d9262408acb9e8872acc31f30d9d9.png")

3. 码云(gitee)涉及图床失效问题，考虑使用其他替代方案：**阿里云的 OSS**

## 三、PicGo配置

1. 安装picgo

   [详见](https://phen.coding.net/p/book-wiki/d/init-linux-env/git/tree/master/install_software.md)

2. 安装gitee插件

   点击左侧的“插件设置”，搜索插件“gitee-uploader”并安装

   ![image-20230113225417605](https://i.328888.xyz/2023/01/13/wx9xU.png "328332649316119-pchen-20230113225417-c4b38d35fb03ec13561250525ae3758e.png")

3. 配置gitee

   - 安装完毕重新启动，点开"图床设置"可以看到 gitee图床，点击“gitee”

     ![image-20230113225507742](https://i.328888.xyz/2023/01/13/wxJLv.png "125209439898660-pchen-20230113225507-92870ec4bd0f177d6e4fb507b768732c.png")

   - 属性选项说明如下

   | 属性   | 说明                       |
   | ------ | -------------------------- |
   | repo   | gitee仓库名称: pymango/oss |
   | token  | 设置的私人令牌             |
   | branch | 代码床分支：默认master     |
   | path   | 图片存储路径：images       |

4. 其他配置

   - 设置PigGo-Server

     ![image-20230113225602288](https://i.328888.xyz/2023/01/13/wxMcy.png "1122001483711418-pchen-20230113225602-2eb65f44111b1551e0fa5323e811a667.png")

   - 打开“时间戳重命名”

     ![image-20230113225644110](https://i.328888.xyz/2023/01/13/wxPo5.png "567532310967114-pchen-20230113225644-96156fdb187c387f51142251c5cc6f61.png")

## 四、Typora配置接入图床

1. 打开`Typora` -> 文件 -> 偏好设置 -> 图像，按照①②③④⑤配置。其中**④**处注意下，路径为picgo命令路径

   ![image-20230113225731223](https://i.328888.xyz/2023/01/13/wx18Z.png "44638456280852-pchen-20230113225731-26b7a1867c9df45308d6dce45485c886.png")

2. 点击⑤处，出现如下结果表示配置成功

   ![image-20230113225851881](https://i.328888.xyz/2023/01/13/wxQFF.png "950404720957574-pchen-20230113225851-c68980fafa0cf10aac9fdfbd45f879b5.png")



# Typora+PyPicGo+Imgloc+百度云盘搭建博客写作环境

## 一、fork Pypicgo源码

[github仓](https://jihulab.com/pystd/pypicgo.git)

## 二、二次开发pypicgo

1. 基于[imgloc api](https://v4-docs.chevereto.com/developer/api/api-v1.html)开发imgloc图床uploader
2. 基于[百度网盘api](https://pan.baidu.com/union/doc/nksg0sbfs)开发百度网盘plugins，用于备份图片
3. 修改配置文件 `~/.PyPicGo/config.yaml`
4. 设置uploader为imgloc且添加BaiduNetdiskPlugin插件

## 三、Typora配置接入图床

配置上传命令为：`/mnt/linux/pyenv/python3.10/bin/python /mnt/linux/apps/PyPicGo/run.py -f`

