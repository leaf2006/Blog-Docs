---
title: 😃 树莓派折腾手册 (一)——准备系统 😃
date: 2021-06-18 19:37:48
categories: 树莓派
tags: [树莓派, Linux系统]
description: 树莓派折腾手册 1 准备系统 😃
index_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619154345.jpg
banner_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619154345.jpg
sticky: 100
---

# 😃 树莓派折腾手册 (一)——准备系统 😃

## 烧录官方`Debian 10 buster`系统镜像：

 先用 **SDFormatter** 格式化一下内存卡叭:

![hZkDB8qxtRgQz3S](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152409.png)

### 1. 烧写镜像用到的软件： **Win32 Disk Image** 

![Zb6CEHnf17oqO5Q](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152403.png)

 - 然后在U盘的根目录建立一个空白的 **ssh文件  方便ssh远程连接** 
   ![FV5qpvWz7LtsOgi](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/sasw.png)
- 用 **Windows PowerShell** 连接树莓派ssh
  `shift+右键` 呼出**Windows PowerShell**
  **完整连接语法**:

```shell
ssh -p 端口号 用户名@主机地址
```

> 树莓派默认的用户名 **pi** 密码 **raspberry**![QLA74lscbwzRWY2](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152448.png)

 - 树莓派扩展TF卡分区:
   sudo raspi-config --> Advanced options -->Expand Filesystem, 确认重启

### 2. 启动树莓派HDMI功能 : 

- 编辑`config.txt`文件，修改以下参数:
  sudo nano /boot/config.txt

  - 把下面#注释符号去掉
    hdmi_force_hotplug=1  #启用HDMI热插拔功能
    config_hdmi_boost=4    #启用加强HDMI信号

  > 不出意外的话应该可以接上，但是我的没有声音输出诶
  > 注：如果还是不能的话，找到#hdmi_group=1这句话，把前面的#注释符号去掉，把数字改成 2强行指定显示器类型：1是连接老式电视，2代表连接新电视。

## 树莓派 `Debian 10 buster` 换清华源：

```shell
sudo nano /etc/apt/sources.list
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib

sudo nano /etc/apt/sources.list.d/raspi.list
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
```

- 更新源列表: `sudo apt-get update`
- 更新软件版本，升级软件包: `sudo apt-get upgrade`

## 树莓派`rasp-config`相关设置

### 1. 设置pi，root用户密码，并解锁:

```shell
#树莓派修改密码，顺便解锁root用户
sudo passwd pi
sudo passwd root
#解锁root用户
sudo passwd --unlock root
#如果想在终端直接用root用户登录，编辑
sudo nano /etc/ssh/sshd_config
修改 PermitRootLogin without-password 为 PermitRootLogin yes
```

 ### 2. **respi本地化**操作 :

 - 安装中文字体，提供几个Linux中文字体库:

 ```shell
sudo apt-get install xfonts-wqy
sudo apt-get install ttf-wqy-zenhei ttf-wqy-microhei
 ```

- 设置终端中文显示: `sudo raspi-config`:
  选择change_locale，在Default locale for the system environment:中选择zh_CN.UTF-8。
  往下翻一会儿直到找到zh_CN UTF-8把光标移动到前面，然后按下空格键打上*
  ![3hQeD8k9L1mgTZc](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152513.png)
- 改变键盘布局: `sudo dpkg-reconfigure keyboard-configuration`

### 3. 设置 vnc桌面 连接

> 注:这里放弃了树莓派自带的 **realvnc** 因为不支持网页 **novnc** 且功能很少，所以用 **Tightvnc** 代替

- 安装**Tightvncserver**: `sudo apt-get install tightvncserver`
- 安装好之后设置一个**VNC密码**:  vncpasswd

> 注: 先输入操作密码两次，然后会询问是否设置一个查看(view-only)密码，按自己喜欢，一般没必要。

- 设置**开机自启动** :

>设置**开机启动**，需要在 **/etc/init.d/** 中创建一个文件。例如**tightvncserver**:  (启动脚本的名称，有和程序名一致的习惯)

```sh
sudo nano /etc/init.d/tightvncserver
# 内容如下:
#!/bin/sh
### BEGIN INIT INFO
# Provides:          tightvncserver
# Required-Start:    $local_fs
# Required-Stop:     $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start/stop tightvncserver
### END INIT INFO

# More details see:
# http://www.penguintutor.com/linux/tightvnc

### Customize this entry
# Set the USER variable to the name of the user to start tightvncserver under
export USER='pi'
### End customization required

eval cd ~$USER

case "$1" in
  start)
    # 启动命令行。此处自定义分辨率、控制台号码或其它参数。
    su $USER -c '/usr/bin/tightvncserver -depth 16 -geometry 800x600 :1'
    echo "Starting TightVNC server for $USER "
    ;;
  stop)
    # 终止命令行。此处控制台号码与启动一致。
    su $USER -c '/usr/bin/tightvncserver -kill :1'
    echo "Tightvncserver stopped"
    ;;
  *)
    echo "Usage: /etc/init.d/tightvncserver {start|stop}"
    exit 1
    ;;
esac
exit 0
```

然后给**tightvncserver文件**加**执行权限**：

```shell
sudo chmod 755 /etc/init.d/tightvncserver
```

并更新**开机启动列表**：

```shell
sudo update-rc.d tightvncserver defaults
```

一些**service命令** :

```shell
sudo service tightvncserver restart #重启服务
sudo service tightvncserver start/stop #关闭/开启服务
sudo service tightvncserver status #查看服务运行状态
```

> 附:   vnc客户端下载
> [vnc官网](https://www.realvnc.com/en/connect/download/viewer/)

连接成功惹~:
![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152521.png)
![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152531.png)
编辑 ./vnc/xstartup 配置文件使其能与windown共享剪贴板

```shell
sudo nano .vnc/xstartup

#在后方追加
vncconfig -nowin -iconic &
#重启Tightvncserver
sudo service tightvncserver restart
```

### 4.部署**novnc网页** :

> 方便在网页上控制树莓派屏幕，但是**不支持realvnc**

- 安装 **git 支持**

```shell
sudo apt-get install git
```

- **克隆** novnc项目:

> 在中国大陆听说~~加上 *https://gproxy.cn* 就可以加速 **克隆** 速度丫~~ 改口 应换成  [https://github.com.cnpmjs.org/](https://github.com.cnpmjs.org/)

```shell
git clone https://github.com/kanaka/noVNC #源地址
git clone https://github.com.cnpmjs.org/kanaka/noVNC #加速地址
```

- 运行 **novnc** 并设置**开机自启动**:

```shell
cd noVNC
# 初始化可能有点点慢
./utils/launch.sh --vnc localhost:5901 #监听5901 vnc端口
```

![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152538.png)

- 尝试访问: [http://raspiberry:6080](http://raspiberry:6080)  可
  ![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152542.png)
- 一些**高级设置** :

```shell
./utils/websockify --web ./ 8787 localhost:5901  #修改6080默认端口
./utils/websockify --web ./ 8787 192.168.1.10:5901 #可以讲localhost改成所有安装了vncserver的IP地址
```

 **连接速度太慢可以安装Python的numpy库解决**

- 设置**开机启动**：

```shell
#编辑开机启动项
sudo nano /etc/rc.local
#以pi用户运行程序
su pi -c "/home/pi/noVNC/utils/launch.sh --vnc localhost:5901" &
```

![fEVci5dlNCMaUs4](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152617.png)

### 5.安装`cockpit` web可视化管理

```shell
sudo apt-get update
sudo apt-get install cockpit
```

> 安装的依赖有  **一丢丢多**

- 默认是用`https`访问，需要修改配置文件使其能`http`访问

```shell
sudo nano /etc/cockpit/cockpit.conf #这个文件默认是不存在的需要新建

[WebService]
AllowUnencrypted=true
LoginTitle=鲸云pi
```

- 配置**开机启动**

```shell
sudo systemctl enable cockpit.socket
sudo systemctl start cockpit.socket
```

- 页面

  ![image-20200805174634421](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152626.png)

## Python设置

### 概况

> 树莓派默认安装了两个版本的**Python**

![](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152632.png)

### 树莓派pip换源

>**pip**更换为**国内源**，可以大大的提高安装成功率和速度。不管你用的是**pip3还是pip，方法都是一样的**

```shell
mkdir ~/.pip
nano ~/.pip/pip.conf

#写入
[global]
timeout = 5000
index-url = https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
[install]
use-mirrors = true
mirrors = https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

- **更新pip版本**

```shell
python3 -m pip install --upgrade pip
```

![seVOiSZrBKgE827](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619152649.png)

- **树莓派指定Python版本安装模块**

```shell
sudo pip3 install XXX   #Python3版本
sudo pip install XXX   #Python2版本
```

## 部署zsh
```shell
sh -c "$(wget -O- https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"

git clone https://github.com.cnpmjs.org/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions

 git clone https://github.com.cnpmjs.org/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
 
 ZSH_DISABLE_COMPFIX=true
```

