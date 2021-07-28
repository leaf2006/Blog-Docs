---
title: KaliLinuxHelp
date: 2021-06-18 22:16:49
categories: 技术
tags: [Linux系统, 渗透]
description: Kali 平台渗透教程
index_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619154844.jpg
banner_img: https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619154844.jpg
sticky: 95
---

![Kali](https://cdn.jsdelivr.net/gh/AdminWhaleFall/Pic@master/img/20210619154844.jpg)

# Kali Linux 漏洞审计 批量扫描 弱密码撞库 教程

# nmap端口扫描

## 安装

`sudo apt-get install nmap`

## 语法

`nmap [扫描类型] [扫描参数] [hosts 地址与范围]`

## 扫描类型

- -sT    TCP connect() 扫描，这是最基本的 TCP 扫描方式。这种扫描很容易被检测到，在目标主机的日志中会记录大批的连接请求以及错误信息。     

- -sS    TCP 同步扫描 (TCP SYN)，因为不必全部打开一个 TCP 连接，所以这项技术通常称为半开扫描 (half-open)。这项技术最大的好处是，很少有系统能够把这记入系统日志。不过，你需要 root 权限来定制 SYN 数据包。     

- -sF,-sX,-sN    秘密 FIN 数据包扫描、圣诞树 (Xmas Tree)、空 (Null) 扫描模式。这些扫描方式的理论依据是：关闭的端口需要对你的探测包回应 RST 包，而打开的端口必需忽略有问题的包（参考 RFC 793 第 64 页）。     

- -sP    ping 扫描，用 ping 方式检查网络上哪些主机正在运行。当主机阻塞 ICMP echo 请求包是 ping 扫描是无效的。nmap 在任何情况下都会进行 ping 扫描，只有目标主机处于运行状态，才会进行后续的扫描。     

- -sU    UDP 的数据包进行扫描，如果你想知道在某台主机上提供哪些 UDP（用户数据报协议，RFC768) 服务，可以使用此选项。     

- -sA    ACK 扫描，这项高级的扫描方法通常可以用来穿过防火墙。     -sW    滑动窗口扫描，非常类似于 ACK 的扫描。     

- -sR    RPC 扫描，和其它不同的端口扫描方法结合使用。     -b    FTP 反弹攻击 (bounce attack)，连接到防火墙后面的一台 FTP 服务器做代理，接着进行端口扫描。    

## 扫描参数

- -P0    在扫描之前，不 ping 主机。    
- -PT    扫描之前，使用 TCP ping 确定哪些主机正在运行。     
- -PS    对于 root 用户，这个选项让 nmap 使用 SYN 包而不是 ACK 包来对目标主机进行扫描。    
- -PI    设置这个选项，让 nmap 使用真正的 ping(ICMP echo 请求）来扫描目标主机是否正在运行。    
- -PB    这是默认的 ping 扫描选项。它使用 ACK(-PT) 和 ICMP(-PI) 两种扫描类型并行扫描。如果防火墙能够过滤其中一种包，使用这种方法，你就能够穿过防火墙。     
- -O    这个选项激活对 TCP/IP 指纹特征 (fingerprinting) 的扫描，获得远程主机的标志，也就是操作系统类型。     
- -I    打开 nmap 的反向标志扫描功能。    
- -f    使用碎片 IP 数据包发送 SYN、FIN、XMAS、NULL。包增加包过滤、入侵检测系统的难度，使其无法知道你的企图。     
- -v    冗余模式。强烈推荐使用这个选项，它会给出扫描过程中的详细信息。     
- -S  <IP>   在一些情况下，nmap 可能无法确定你的源地址 (nmap 会告诉你）。在这种情况使用这个选项给出你的 IP 地址。    
- -g port    设置扫描的源端口。一些天真的防火墙和包过滤器的规则集允许源端口为 DNS(53) 或者 FTP-DATA(20) 的包通过和实现连接。显然，如果攻击者把源端口修改为 20 或者 53，就可以摧毁防火墙的防护。     
- -oN    把扫描结果重定向到一个可读的文件 logfilename 中。    
- -oS    扫描结果输出到标准输出。    
- --host_timeout    设置扫描一台主机的时间，以毫秒为单位。默认的情况下，没有超时限制。     
- --max_rtt_timeout    设置对每次探测的等待时间，以毫秒为单位。如果超过这个时间限制就重传或者超时。默认值是大约 9000 毫秒。    
- --min_rtt_timeout    设置 nmap 对每次探测至少等待你指定的时间，以毫秒为单位。    
- -M count    置进行 TCP connect() 扫描时，最多使用多少个套接字进行并行的扫描。 

## 端口状态

- Open（开放的）意味着目标机器上的应用程序正在该端口监听连接 / 报文。
- filtered（被过滤的） 意味着防火墙，过滤器或者其它网络障碍阻止了该端口被访问，Nmap 无法得知 它是 open（开放的） 还是 closed（关闭的）。  
- closed（关闭的） 端口没有应用程序在它上面监听，但是他们随时可能开放。 
- unfiltered（未被过滤的）当端口对 Nmap 的探测做出响应，但是 Nmap 无法确定它们是关闭还是开放时 open filtered 开放或者被过滤的 closed filtered 关闭或者被过滤的

## 扫描实例

> 阿里云服务器网段大全：[CSDN博客](https://blog.csdn.net/nxuu01/article/details/102779652) [CSDN-eagle_min大佬](https://blog.csdn.net/eagle_min/article/details/82260622)
>
> 腾讯云ip段：[CSDN-eagle_min大佬](https://blog.csdn.net/eagle_min/article/details/82260611)

1. 用nmap扫描该网段开放的888端口保存在pma.txt文件 格式要求：ip:port
   `nmap -vv -n --open -p 888 网段 | awk -F'[ /]' '/Discovered open port/{print $NF":"$4}' >> pma.txt`

# Hydra 弱密码破解

## 安装

`sudo apt-get install hydra`

> Hydra是一款**非常强大的暴力破解工具**，它是由著名的黑客组织THC开发的一款**开源暴力破解工具**。Hydra是一个验证性质的工具，主要目的是：展示安全研究人员从**远程获取一个系统认证权限**。

## 常见参数

-  -R: 继续从上一次进度接着破解
- -S: 大写，采用SSL链接
- -s <PORT>： 小写，可通过这个参数指定非默认端口
- -l <LOGIN>： 指定破解的用户，对特定用户破解
- -L <FILE>： 指定用户名字典
- p <PASS>： 小写，指定密码破解，少用，一般是采用密码字典
- -P <FILE>： 大写，指定密码字典
- -e <ns>： 可选选项，n：空密码试探，s：使用指定用户和密码试探
- -C <FILE>： 使用冒号分割格式，例如“登录名:密码”来代替 -L/-P 参数
- -M <FILE>： 指定目标列表文件一行一条
- o <FILE>： 指定结果输出文件
-  -f ： 在使用-M参数以后，找到第一对登录名或者密码的时候中止破解
-  -t <TASKS>： 同时运行的线程数，默认为16
-  -w <TIME>： 设置最大超时的时间，单位秒，默认是30s
-  -v / -V： 显示详细过程
-  server： 目标ip
-  service： 指定服务名，支持的服务和协议
-  OPT： 可选项

---------

## 实例分析

### 1.破解ssh

`hydra -M sship.txt ssh -L user.txt -P passwd.txt -e ns -f -vV -t 4 -o ssh.txt`

> -M 指定目标列表文件 一条一行
>
> -L 指定用户字典 
>
> -P 指定密码字典 
>
> -e ns 空密码试探
>
> -f 当破解一个成功密码就停止
>
> -o 把成功的输出到ssh.txt文件 
>
> -vV 显示详细信息

### 2.破解3389远程登录

`hydra -M rdpip.txt rdp -L user.txt -P passwd.txt -e ns -f -vV -t 16 -o rdp.txt`

### 3.破解MySQL数据库

`hydra -M mysqlip.txt rdp -L user.txt -P passwd.txt -e ns -f -vV -t 16 -o mysql.txt`

