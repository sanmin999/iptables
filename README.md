Debian安装配置Iptables防火墙
===========
服务器通常会安装防火墙,Debian上有很防火墙,Iptables为比较常用的免费防火墙,Iptables能够提供数据包过滤,网络地址转换(NAT)等功能.
在Debian上手工配置Iptables的资料比较少,本文做一个详细的介绍.

第一步,首先确定你的系统已经安装Iptables.打开SSH终端,输入
```Bash
whereis iptables
```
如果能看到如下类似信息,说明你已经安装了iptables
```Bash
iptables: /sbin/iptables /usr/share/iptables /usr/share/man/man8/iptables.8.gz
```
如果不是这个提示,或者没有任何提示,那你的Debian上可能没有安装iptables
请使用如下命令安装:
```Bash
sudo apt-get install iptables
```
注意:本文所有命令在普通帐号下完成,本普通帐号使用sudo具有root权限,本人不建议直接使用root用户

第二步:查看Iptables目前的配置信息
可以使用如下命令查看
```Bash
sudo iptables -L
```
如果你是第一次安装配置iptables,你可能会看到如下结果:
```Bash
Chain INPUT (policy ACCEPT)
target prot opt source destination
Chain FORWARD (policy ACCEPT)
target prot opt source destination
Chain OUTPUT (policy ACCEPT)
target prot opt source destination
```
这个结果,也就是防火墙充许所有的请求,就如没有设置防火墙一样.

第三步:配置Iptables
配置Iptables,我们先把一个基本的Iptables的规则文章保存起来,这个规则文章做为测试用
```Bash
sudo vim /etc/iptables.test.rules
```
然后在这个文章中输入如下规则内容,这个内容是debian官方给出的基本配置
```Bash
*filter

# Allows all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0

-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT
# Accepts all established inbound connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
# Allows all outbound traffic
# You could modify this to only allow certain traffic
-A OUTPUT -j ACCEPT
# Allows HTTP and HTTPS connections from anywhere (the normal ports for websites)
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT
# Allows SSH connections for script kiddies
# THE -dport NUMBER IS THE SAME ONE YOU SET UP IN THE SSHD_CONFIG FILE
-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
# Now you should read up on iptables rules and consider whether ssh access
# for everyone is really desired. Most likely you will only allow access from certain IPs.
# Allow ping
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
# log iptables denied calls (access via 'dmesg' command)
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7
# Reject all other inbound - default deny unless explicitly allowed policy:
-A INPUT -j REJECT
-A FORWARD -j REJECT
COMMIT
```

保存本文件,然后把本规则加载,使之生效,注意,iptables不需要重启,加载一次规则就成了
```Bash
sudo iptables-restore < /etc/iptables.test.rules
```
然后再查看最新的配置,应该所有的设置都生效了.
```Bash
sudo iptables -L
```
第四步:保存生效的配置,让系统重启的时候自动加载有效配置
iptables提供了保存当前运行的规则功能
```Bash
iptables-save > /etc/iptables.up.rules
```
注意,如果当前用户不是root,即使使用了sudo,也会提示你没有权限,无法保存,所以执行本命令,你必须使用root用户.
可以使用sudo -i快速转到root,使用完成,请及时使用su username切换到普通帐户.

为了重启服务器后,规则自动加载,我们创建如下文件:
```Bash
sudo vim /etc/network/if-pre-up.d/iptables
```
本文章的内容如下:
```Bash
#!/bin/bash
/sbin/iptables-restore < /etc/iptables.up.rules
```
最后,设置本文章具体可执行仅限
```Bash
chmod +x /etc/network/if-pre-up.d/iptables
```
第五:其它
如果你想设置某ip段可以访问所有服务,你需要在iptables.test.rules文件中加入
```Bash
-A INPUT -m iprange --src-range 192.168.1.1-192.168.1.199 -j ACCEPT
```
然后从第三步再设置一次.注意iptables.test.rules不是必须的,它只是让你的修改时,能更好的测试.

转载至：
"夜世界" http://www.voland.com.cn/category/technique/config/page/2 
