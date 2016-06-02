# Cobbler 自动化系统部署
自动化操作系统部署Cobbler
###Cobbler简介
&#160;&#160;&#160;&#160;&#160;&#160;&#160;Cobbler 可以用来快速建立 Linux 网络安装环境，它已将 Linux 网络安装的技术门槛，从大专以上文化水平，成功降低到初中以下，连补鞋匠都能学会。
#####摘自百度百科
&emsp;&emsp;网络安装服务器套件 Cobbler(补鞋匠)从前，我们一直在做装机民工这份很有前途的职业。自打若干年前 Red Hat 推出了 Kickstart，此后我们顿觉身价倍增。不再需要刻了光盘一台一台地安装 Linux，只要搞定 PXE、DHCP、TFTP，还有那满屏眼花缭乱不知所云的 Kickstart 脚本，我们就可以像哈里波特一样，轻点魔棒，瞬间安装上百台服务器。这一堆花里胡哨的东西可不是一般人都能整明白的，没有大专以上学历，通不过英语四级， 根本别想玩转。总而言之，这是一份多么有前途，多么有技术含量的工作啊。很不幸，Red Hat 最新（Cobbler项目最初在2008年左右发布）发布了网络安装服务器套件 Cobbler(补鞋匠)，它已将 Linux 网络安装的技术门槛，从大专以上文化水平，成功降低到初中以下，连补鞋匠都能学会。对于我们这些在装机领域浸淫多年，经验丰富，老骥伏枥，志在千里的民工兄弟们来说，不啻为一个晴天霹雳。
###环境准备
<pre>
1. 查看系统版本
[root@linux-node1 ~]# cat /etc/redhat-release 
CentOS Linux release 7.2.1511 (Core) 
2. 内核版本
[root@linux-node1 ~]# uname -a
Linux linux-node1 3.10.0-327.18.2.el7.x86_64 #1 SMP Thu May 12 11:03:55 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
3. 确保selinux关闭
[root@linux-node1 ~]# getenforce
Disabled
4. iptables 关闭
[root@linux-node1 ~]# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
5. 主机名
[root@linux-node1 ~]# hostname
linux-node1
6. hosts本地解析
[root@linux-node1 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.56.11  linux-node1 linux-node1.example.com
192.168.56.12  linux-node2 linux-node2.example.com
7. 安装yum源
[root@linux-node1 ~]# rpm -ivh http://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
</pre>

###安装Cobbler
<pre>
[root@linux-node1 ~]# yum install cobbler cobbler-web pykickstart httpd dhcp tftp xinetd -y
包说明：
cobbler          #Cobbler程序包
cobbler-web      #Cobbler的web服务包
pykickstart      #Cobbler检查kickstart语法错误
httpd            #Apache web服务
dhcp             #Dhcp服务
tftp             #Tftp服务
</pre>







