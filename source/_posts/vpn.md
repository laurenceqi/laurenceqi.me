---
layout: post
title: 如何在CentOS搭建PPTP VPN
categories: 
- 网络
tags:
- pptp
---

VPN(virtual private network)是虚拟专用网，是指通过公网建立虚拟的专用网络。VPN一般通过隧道协议、消息加密等建立一个虚拟的点对对连接。因此VPN可以用来使用户通过外网访问企业内网、通过一台中继机器就可访问相互隔离的网络等等。在天朝因为有GFW的存在，所以很多用户也是用VPN来访问墙外的网络。

实现VPN由很多种协议，l2tp、pptp、openVPN等。PPTP虽然安全性较差，但是建立快速、移动设备可用、运行快速且资源占用较少。以Centos为例，说明一下建立PPTP VPN的步骤。

### Step1 安装PPTP
1. 安装EPEL YUM源

        rpm -ivh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm

2. 安装pptpd  
        
        yum install pptp
        
3. 修改pptpd配置
    
    编辑/etc/pptpd.conf，增加以下配置：
            
        localip 192.168.0.1
        remoteip 192.168.0.204-238
    
    localip代表虚拟专用网中的本机地址，remoteip每一个连接上来的客户端会分配一个remoteip
4. 为pptp设置认证，即登陆的用户名和密码
    修改/etc/ppp/chap-secrets文件，增加以下内容：
    
        # Secrets for authentication using CHAP
        # client        server  secret                  IP addresses
        username pptpd password *
    
    username、password替换为用户名及密码；当前由pptpd服务，所以server选择pptpd， IP addresses输入*代表任何连入IP

### Step2 设置DNS
1. 编辑/etc/ppp/options.pptpd,增加以下内容
    
        ms-dns 8.8.8.8
        ms-dns 8.8.4.4
        
2. 重启pptpd服务，使配置生效
        
        service pptpd restart
        
### Step3 设置网络转发
1. 修改/etc/sysctl.conf，增加如下内容
         
         net.ipv4.ip_forward = 1
         
2. 运行以下命令，使配置生效

        sysctl -p
        
### Step4 设置iptables
1. 运行以下命令，创建一条NAT规则

        iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE && iptables-save           
        
### Step5 设置客户端     
1. 以ios为例，在vpn中选择增加，类型为PPTP，服务器为服务器地址，用户名、密码为步骤1.4中设置的username、password，保存。
2. 开启VPN，即可发现已成功建立连接，enjoy！  

        