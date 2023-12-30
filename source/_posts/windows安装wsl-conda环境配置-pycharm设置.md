---
title: windows安装wsl+conda环境配置+pycharm设置
date: 2023-12-30 21:57:51
categories: 其他
---

## 一. windows 安装 wsl

> https://learn.microsoft.com/zh-cn/windows/wsl/install

1. 以**管理员**身份运行 Windows PowerShell

   执行：

   ```shell
   wsl --install
   ```

   出现以下提示表示安装成功：

   ![windows安装wsl-1](windows安装wsl-1.png)

2. 重启电脑，自动弹出子系统启动界面

   默认为 ubuntu

   ![windows安装wsl-2](windows安装wsl-2.png)

3. 设置用户名密码

   ![windows安装wsl-3](windows安装wsl-3.png)

   完毕

   > 默认安装在 C 盘，如果想移动到其他盘，参考：
   >
   > https://blog.csdn.net/yihuajack/article/details/119915303

<br />

## 二. conda 环境设置

1. apt-get 替换国内镜像源

   > https://blog.csdn.net/qq_21095573/article/details/99736630
   >
   > https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.3e221b11fHrGAW

   进入目录`/etc/apt/`：

   ```shell
   cd /etc/apt/
   ```

   备份当前`sources.list`：

   ```shell
   sudo cp sources.list sources.list.copy
   ```

   按照国内镜像源配置说明更改`sources.list`内容：

   ![conda环境设置-1](conda环境设置-1.png)

   更换完后，执行：

   ```shell
   sudo apt-get update
   ```

   如果没有出现异常则表示更换完毕

2. 安装 gcc 环境

   ```shell
   sudo apt install gcc build-essential
   ```

3. 安装 conda

   > https://blog.csdn.net/weixin_44159487/article/details/105620256

   推荐安装 miniconda

   ```shell
   # 返回用户目录
   cd ~
   # 安装miniconda包
   wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-latest-Linux-x86_64.sh
   # 执行sh文件
   bash Miniconda3-latest-Linux-x86_64.sh
   ```

   按照提示安装即可

   安装完毕后重启终端，如果能看到（base）

   ![conda环境设置-2](conda环境设置-2.png)

   表示安装成功

4. 更换 conda 的 pip 安装源

   > https://www.cnblogs.com/137point5/p/15000954.html

   永久更换清华源

   ```shell
   pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
   ```

   ![conda环境设置-2](conda环境设置-3.png)

   但是个人更喜欢配置多个源，根据上图可以看到 pip 的配置文件存放在`/home/当前用户/.config/pip/pip.conf`中

   修改该文件：

   ```ini
   [global]
   index-url=https://pypi.tuna.tsinghua.edu.cn/simple/
   extra-index-url=http://mirrors.aliyun.com/pypi/simple/

   [install]
   trusted-host=pypi.tuna.tsinghua.edu.cn
   	mirrors.aliyun.com
   ```

   > 这里注意不要配置官方源，否则默认会从官方源下载，导致下载速度还是很慢

5. 测试一下

   创建虚拟环境

   ```shell
   conda create -n test python=3.10
   ```

   切换到`test`环境

   ```shell
   conda activate test
   ```

   尝试安装一些包

   ![conda环境设置-4](conda环境设置-4.png)

   完毕

<br />

## 三. pycharm 设置

前提！前提！前提！需要 pycharm 专业版！！！如果不是专业版无法连接到 wsl 中的 conda 环境

![pycharm-1](pycharm-1.png)

解释器设置选择 WSL

![pycharm-2](pycharm-2.png)

后续操作同主系统直接使用 conda

![pycharm-3](pycharm-3.png)
