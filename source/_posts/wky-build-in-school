---
title: 学校玩转玩客云手册
date: 2021-12-19 19:37:48
categories: 玩客云
tags: [玩客云, Linux系统]
description: 记一次在学校一体机配置玩客云的经历
index_img: 
banner_img: 
sticky: 109
---
# 学校玩客云修复计划

## 重新烧写玩客云

1. 带螺丝批，拆机短接，**双公 `USB`** 连接一体机烧写 `Armbian5.67直刷包带宝塔.7z`

2. 玩客云通过网线直连一体机，参考：[树莓派使用网线直连电脑的方法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/37761024)

   - 手机开热点一体机连接

   - Win10可以直接从【设置->网络和Internet->状态->更改适配器设置】进入可以看到，我们的本地网络连接方式有 **WLAN无线连接** 和 **以太网有线连接** 两种方式。

   - 记录当前我们的网络连接状况：`arp -a `

   - **共享WLAN网络给以太网**

     更改适配器设置 界面中选择修改 WLAN 属性。选择共享，设置共享网络给以太网。（其他选项全部选择）

   - **查询树莓派的IP地址**

     将树莓派的网线插到一体机的PC端口后再查询一次：`arp -a`

3. 利用 **Putty** 连接玩客云。

   SSH远程账号 `root`  ：密码1234
   宝塔账号 `onecloud`  ：密码123456

## 玩客云配置

1. **换源** 

   > 参考：[ 玩客云刷armbian更新源报错The repository ‘http://apt.armbian.com stretch Release‘ does not have a Release file](https://blog.csdn.net/qq_42877824/article/details/119332805)

   **修改源配置文件**
   
   ```shell
   sudo nano /etc/apt/sources.list
   
   deb http://mirrors.ustc.edu.cn/debian stretch main contrib non-free
   deb http://mirrors.ustc.edu.cn/debian stretch-updates main contrib non-free
   deb http://mirrors.ustc.edu.cn/debian stretch-backports main contrib non-free
   deb http://mirrors.ustc.edu.cn/debian-security/ stretch/updates main contrib non-free
   
   sudo nano /etc/apt/sources.list.d/armbian.list
   deb [trusted=yes] http://apt.armbian.com bionic main bionic-utils bionic-desktop
   
   # 更新源
   sudo apt update
   sudo apu upgrade
   ```
   
2. **卸载自带的宝塔面板**

   ```shell
   apt-get install wget git nginx -y
   wget http://download.bt.cn/install/bt-uninstall.sh
   sh bt-uninstall.sh
   ```

3. **部署 `Frpc` 内网穿透** 

   **下载并解压**

   ```shell
   cd ~
   wget https://hub.fastgit.org/fatedier/frp/releases/download/v0.38.0/frp_0.38.0_linux_arm.tar.gz
   tar -xzvf frp_0.38.0_linux_arm.tar.gz
   ```

   **拷贝配置文件：**

   ```ini
   [common]
   server_addr = frps.420400150.xyz
   server_port = 7000
   token = 12345678
   user = whalefall
   login_fail_exit = false
   protocol = tcp
   tcp_mux = true
   dns_server = 114.114.114.114
   tls_enable = true
   
   [ssh_tcp]
   type = tcp
   local_ip = 127.0.0.1
   local_port = 22
   remote_port = 42202
   use_encryption = false
   use_compression = true
   
   [pi_vpn]
   type = tcp
   local_ip = 127.0.0.1
   local_port = 1194
   remote_port = 41194
   use_encryption = false
   use_compression = true
   ```

   **配置 systemctl 服务并设置开机自启动**

   systemctl 配置文件夹在 `/etc/systemd/system/frpc.service` 目录下

   配置文件：

   ```ini
   [Unit]
   Description=Frp Client Service
   After=network.target
   
   [Service]
   Type=simple
   User=root
   Restart=on-failure
   RestartSec=5s
   ExecStart=/root/frp/frpc -c /root/frp/frpc.ini
   ExecReload=/root/frp/frpc reload -c /root/frp/frpc.ini
   LimitNOFILE=1048576
   
   [Install]
   WantedBy=multi-user.target
   ```

   启动：

   ```shell
   sudo systemctl start frpc	# 开启
   sudo systemctl status frpc  # 查看状态
   sudo systemctl enable frpc  # 设置开机自启
   ```

4. **Python** 调优

   **Python** 更换国内 `pip` 源：

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
   
   python3 -m pip install --upgrade pip  # 更新pip
   pip3 install httpx  # 测试
   ```

   

5. 部署**校园网自动登录 `FRPC` 配置自动获取脚本**

   > 项目地址：[AdminWhaleFall/rpi-ping: 树莓派自动上传信息工具. (github.com)](https://github.com/AdminWhaleFall/rpi-ping)

   ```shell
   cd ~
   git clone https://hub.fastgit.org/AdminWhaleFall/rpi-ping
   chmod 777 -R rpi-ping
   
   cd rpi-ping
   pip3 install -r req... (Tab补全)
   python3 main.py  # 测试运行
   ```

   脚本设置为 systemctl 服务：

   新建 `/etc/systemd/system/rpi.service` ：

   ```ini
   [Unit]
   Description=Python Task Service
   After=network.target
   
   [Service]
   Type=simple
   User=root
   Group=root
   ExecStart=/usr/bin/python3 /root/rpi-ping/main.py
   Restart=always
   RestartSec=2
   PrivateTmp=true
   
   [Install]
   WantedBy=multi-user.target
   ```

## 玩客云网络配置（重要）

1. 设置静态ip：

   > 我们学校的校园网没有 dhcp 服务器，要手动设置IP，教学楼的网段是：**192.168.5.0/24**

   ```shell
   nano /etc/network/interfaces
   
   # 注释掉
   # iface eth0 inet dhcp
   # 添加静态IP
   iface eth0 inet static
   address 192.168.5.12
   netmask 255.255.255.0
   gateway 192.168.5.1
   dns-nameservers 114.114.114.114 192.168.3.3
   
   # address: 地址；netmask: 子网掩码；gateway:路由
   systemctl restart NetworkManager  # 重启网络服务
   ```

   然后把玩客云插到讲台下面的网线处.访问 `192.168.5.12` 测试

2. 永久修改dns：

   禁用 dhcp 分配的dns服务器：

   ```shell
   nano /etc/dhcp/dhclient.conf
   
   # 最后添加一行
   supersede domain-name-servers 114.114.114.114, 8.8.8.8;
   dhclient
   ```

   在 Resolvconf 中设置永久 DNSNameservers ：

   ```shell
   nano /etc/resolvconf/resolv.conf.d/head
   # 添加
   nameserver 114.114.114.114
   
   systemctl restart resolvconf.service
   resolvconf -u
   sudo dpkg-reconfigure resolvconf
   
   cat /etc/resolv.conf
   ```
