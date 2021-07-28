---
title: 😏 树莓派折腾手册（三）——搭建离线下载器 👀
date: 2021-06-18 19:37:48
categories: 树莓派
tags: [树莓派, Linux系统]
description: 树莓派折腾手册 3——搭建离线下载器 👀
index_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619154345.jpg
banner_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619154345.jpg
sticky: 99
---

# 😏 树莓派折腾手册（三）——搭建离线下载器 👀

## 1.挂载硬盘/U盘

>**注意：开始之前先把存储设备格式化成** **fat32文件系统** 

- 树莓派4B Debian10系统插上储存设备后默认自动挂载到  `/media` 目录我们先卸载U盘：

  查询硬盘状态:   `sudo fdisk -l` 

  ![9XBTemAzD6R8yot](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619153119.png)

```shell
sudo umount /media/pi/PI   #这里不能照抄命令，要根据实际情况更改 

#如果出现 target is busy 的情况，我们要强行结束U盘目录下的进程 
sudo fuser -mv -k /media/U盘 名字    # 然后再执行umount卸载命令 
```

- 编辑`/etc/fstab`中添加像下面这样的挂载配置： 

```
sudo nano /etc/fstab

/dev/sda1 /home/pi/disk vfat utf8,uid=1000,gid=1000,umask=000 0 0 #挂载点需要自行更改 sda1要加数字

sudo reboot

mount /dev/sda1 /home/pi/disk -o utf8,uid=1000,gid=1000,umask=000 -t vfat
```

- 重启 ，不出意外的话开机 应该 会自动挂载，且有写权限，用户是pi：如图挂载至 `/home/pi/disk` 目录，有**777权限**

  ![3ilg9S1UJ2EcHPV](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619153142.png)

## 2.部署Aria2离线下载器 

- 下载安装`Aria2`:

```
sudo apt-get update
sudo apt-get install aria2
```

- 安装nginx：

  > 上面已经安装过的**小可爱**可以**跳过**

```
sudo apt-get install nginx
```

- 配置Aria2， 创建配置文件： 

  ```
  #创建目录 
  sudo mkdir /etc/aria2/ 
  #创建配置文件 
  sudo touch /etc/aria2/aria2.conf 
  #创建aria2用户 
  sudo useradd -M -s /usr/sbin/nologin aria2 
  #创建session文件，用于保存进度: 
  sudo touch /etc/aria2/aria2.session 
  #修改文件拥有者为aria2： 
  sudo chown aria2 /etc/aria2 /etc/aria2/aria2.session
  ```

- 编辑`Aria2配置`文件:

  ```
  #根据需要编辑文件: 
  sudo nano /etc/aria2/aria2.conf 
  #配置实例
  
  #默认下载位置，需要改这里！！ 
  dir=/home/pi/disk 
  #断点续传 
  continue=true 
  min-split-size=10M 
  input-file=/etc/aria2/aria2.session 
  save-session=/etc/aria2/aria2.session 
  enable-rpc=true 
  rpc-allow-origin-all=true 
  #只让本机访问6800端口，因为下面让nginx代理 
  rpc-listen-all=false 
  #rpc秘钥，需要改这里 
  rpc-secret=123456 
  rpc默认端口为6800
  #rpc-listen-port=6800 
  listen-port=51413  
  enable-dht=false 
  enable-peer-exchange=false 
  peer-id-prefix=-TR2770- 
  user-agent=Transmission/2.77 
  seed-ratio=0 
  bt-seed-unverified=true 
  bt-save-metadata=true
  ```

- 创建**systemd**文件在 `/lib/systemd/system/aria2.service` 为如下: 

  ```
  sudo nano /lib/systemd/system/aria2.service
  
  #配置实例 
  [Unit] 
  Description=Aria2c download manager 
  After=network.target  
  [Service] 
  Type=simple 
  User=aria2 
  ExecStart=/usr/bin/aria2c  --conf-path=/etc/aria2/aria2.conf  [Install] 
  WantedBy=multi-user.target 
  ```

- 启动**Aria2**:

  ```
  #启动Aria2 
  sudo systemctl start aria2.service 
  #可以设置开机启动 
  sudo systemctl enable aria2.service 
  #如果要关闭开机启动 
  sudo systemctl disable aria2.service
  ```

- 配置**nginx+ariaNg**可视化管理页面：

  > 到[**AriaNG**开源项目页面](https://github.com/mayswind/AriaNg/releases) 获取最新版版本

  ![UO7c5EItjBHRsgN](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619153158.png)

  **把源码克隆到** `/website/AriaNg/`：

  ```shell
  sudo mkdir /website
  sudo chmod 777 -R /website
  cd /website
  mkdir AriaNg
  cd AriaNg
  
  #自己复制最新版链接
  wget https://github.com/mayswind/AriaNg/releases/download/1.1.6/AriaNg-1.1.6.zip
  #祖国加速通道
  wget https://gproxy.cn/https://github.com/mayswind/AriaNg/releases/download/1.1.6/AriaNg-1.1.6.zip
  #解压
  unzip AriaNg-1.1.6.zip
  rm AriaNg-1.1.6.zip
  #授权
  sudo chmod 777 -R /website
  ```

> 为了方便使用，我把**AriaNg**和**jsonrpc**都配置在了 **80 端口**，利用nginx的**代理功能**，把本机 6800 端口隐藏,对外**只暴露 80 端口.** 

```shell
#修改nginx配置文件 
sudo nano /etc/nginx/sites-enabled/default

#添加配置aria2Ng 
location /aria2 { 
            alias /website/AriaNg/; 
            index index.html; 
    } 
#代理jsonrpc 
location /jsonrpc { 
       proxy_pass http://localhost:6800/jsonrpc; 
            proxy_redirect off; 
            proxy_set_header        X-Real-IP       $remote_addr; 
            proxy_set_header        X-Forwarded-For     $proxy_add_x_forwarded_for; 
            proxy_set_header Host $host; 
            #以下代码使支持WebSocket 
            proxy_http_version 1.1; 
            proxy_set_header Upgrade $http_upgrade; 
            proxy_set_header Connection "upgrade"; 
} 

#最后别忘记重启nginx 
sudo systemctl restart nginx 
```

- 尝试访问 [http://树莓派ip/aria2](http://树莓派ip/aria2) ，**设置参数**

- ![wNbqhBl7KWL6Jcx](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619153219.png)

![2o7a8c3qpFODP4Y](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619153228.png)