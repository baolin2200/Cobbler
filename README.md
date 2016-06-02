# Cobbler 自动化系统部署
自动化操作系统部署Cobbler

>注：本文内容参考自互联网，版权归属于互联网，感谢老男孩、徐布斯
###1.1 Cobbler简介
&emsp;&emsp;Cobbler 可以用来快速建立 Linux 网络安装环境，它已将 Linux 网络安装的技术门槛，从大专以上文化水平，成功降低到初中以下，连补鞋匠都能学会。

&emsp;&emsp;Cobbler是一个Linux服务器安装的服务，可以通过网络启动(PXE)的方式来快速安装、重装物理服务器和虚拟机，同时还可以管理DHCP，DNS等。Cobbler可以使用命令行方式管理，也提供了基于Web的界面管理工具(cobbler-web)，还提供了API接口，可以方便二次开发使用。
#####摘自百度百科
&emsp;&emsp;网络安装服务器套件 Cobbler(补鞋匠)从前，我们一直在做装机民工这份很有前途的职业。自打若干年前 Red Hat 推出了 Kickstart，此后我们顿觉身价倍增。不再需要刻了光盘一台一台地安装 Linux，只要搞定 PXE、DHCP、TFTP，还有那满屏眼花缭乱不知所云的 Kickstart 脚本，我们就可以像哈里波特一样，轻点魔棒，瞬间安装上百台服务器。这一堆花里胡哨的东西可不是一般人都能整明白的，没有大专以上学历，通不过英语四级， 根本别想玩转。总而言之，这是一份多么有前途，多么有技术含量的工作啊。很不幸，Red Hat 最新（Cobbler项目最初在2008年左右发布）发布了网络安装服务器套件 Cobbler(补鞋匠)，它已将 Linux 网络安装的技术门槛，从大专以上文化水平，成功降低到初中以下，连补鞋匠都能学会。对于我们这些在装机领域浸淫多年，经验丰富，老骥伏枥，志在千里的民工兄弟们来说，不为一个晴天霹雳。
###1.2 环境准备
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

###1.3 安装Cobbler
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
###1.4 重要配置文件注释：
<pre>
[root@linux-node1 ~]# rpm -ql cobbler
/etc/cobbler                  # 配置文件目录
/etc/cobbler/settings         # cobbler主配置文件，这个文件是YAML格式，Cobbler是python写的程序。
/etc/cobbler/dhcp.template    # DHCP服务的配置模板
/etc/cobbler/tftpd.template   # tftp服务的配置模板
/etc/cobbler/rsync.template   # rsync服务的配置模板
/etc/cobbler/iso              # iso模板配置文件目录
/etc/cobbler/pxe              # pxe模板文件目录
/etc/cobbler/power            # 电源的配置文件目录
/etc/cobbler/users.conf       # Web服务授权配置文件
/etc/cobbler/users.digest     # 用于web访问的用户名密码配置文件
/etc/cobbler/dnsmasq.template # DNS服务的配置模板
/etc/cobbler/modules.conf     # Cobbler模块配置文件
/var/lib/cobbler              # Cobbler数据目录
/var/lib/cobbler/config       # 配置文件
/var/lib/cobbler/kickstarts   # 默认存放kickstart文件
/var/lib/cobbler/loaders      # 存放的各种引导程序
/var/www/cobbler              # 系统安装镜像目录
/var/www/cobbler/ks_mirror    # 导入的系统镜像列表
/var/www/cobbler/images       # 导入的系统镜像启动文件
/var/www/cobbler/repo_mirror  # yum源存储目录
/var/log/cobbler              # 日志目录
/var/log/cobbler/install.log  # 客户端系统安装日志
/var/log/cobbler/cobbler.log  # cobbler日志
</pre>

###1.5 启动服务
<pre>
cobbler的运行依赖于dhcp、tftp、rsync及dns服务。
[root@linux-node1 ~]# systemctl start httpd
[root@linux-node1 ~]# systemctl start cobblerd
[root@linux-node1 ~]# netstat -lntup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3182/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1879/master         
tcp        0      0 127.0.0.1:25151         0.0.0.0:*               LISTEN      4707/python2        
tcp6       0      0 :::80                   :::*                    LISTEN      4678/httpd          
tcp6       0      0 :::22                   :::*                    LISTEN      3182/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      1879/master         
tcp6       0      0 :::443                  :::*                    LISTEN      4678/httpd          
udp        0      0 0.0.0.0:35847           0.0.0.0:*                           4122/dhclient       
udp        0      0 0.0.0.0:68              0.0.0.0:*                           4122/dhclient       
udp6       0      0 :::11445                :::*                                4122/dhclient 
</pre>

###1.6 检查Cobbler的配置，如果看不到下面的结果，再次执行systemctl start cobblerd
<pre>
[root@linux-node1 ~]# cobbler check
The following are potential configuration items that you may want to fix:
1 : The 'server' field in /etc/cobbler/settings must be set to something other than localhost, or kickstarting features will not work.  This should be a resolvable hostname or IP for the boot server as reachable by all machines that will use it.
2 : For PXE to be functional, the 'next_server' field in /etc/cobbler/settings must be set to something other than 127.0.0.1, and should match the IP of the boot server on the PXE network.
3 : change 'disable' to 'no' in /etc/xinetd.d/tftp
4 : some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, elilo.efi, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.
5 : enable and start rsyncd.service with systemctl
6 : debmirror package is not installed, it will be required to manage debian deployments and repositories
7 : The default password used by the sample templates for newly installed machines (default_password_crypted in /etc/cobbler/settings) is still set to 'cobbler' and should be changed, try: "openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'" to generate new one
8 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them
Restart cobblerd and then run 'cobbler sync' to apply changes.
针对上方问题的逐一解决：
1. 修改/etc/cobbler/settings文件中的server参数的值为提供cobbler服务的主机相应的IP地址或主机名"如server: 10.0.0.101"; 此命令可以更改两个参数解决 1，2问题。
sed -i 's/server: 127.0.0.1/server: 192.168.56.11/' /etc/cobbler/settings
检查:
[root@linux-node1 ~]# grep 192.168.56.11  /etc/cobbler/settings
next_server: 192.168.56.11
server: 192.168.56.11
2. 见上条命令结果检查;
3. 修改/etc/xinetd.d/tftp中的disable的参数修改为"disable                 = no";
4. 按照提示执行"cobbler get-loaders"下载loaders;
[root@linux-node1 ~]# cobbler get-loaders
task started: 2016-06-02_223401_get_loaders
查看下载的内容：
[root@linux-node1 ~]# ls /var/lib/cobbler/loaders/
COPYING.elilo  COPYING.syslinux  COPYING.yaboot  elilo-ia64.efi  grub-x86_64.efi  grub-x86.efi  menu.c32  pxelinux.0  README  yaboot
5. 安装提示执行 "systemctl enable rsyncd";
[root@linux-node1 ~]# systemctl enable rsyncd
6. 待定;
7. 创建默认网页用户及密码：
[root@linux-node1 ~]# openssl passwd -1 -salt 'admin' 'password'
$1$admin$mZhVCYpQb/nUmzdFCQFBs0
修改密码为创建的加密码：
[root@linux-node1 ~]# grep default_password /etc/cobbler/settings
default_password_crypted: "$1$admin$mZhVCYpQb/nUmzdFCQFBs0"
8. 待定;
重启cobblerd后再做检查：
还差两项，一个为debian系统相关，一个为电源管理设备相关，此处暂不做调整;
[root@linux-node1 ~]# systemctl restart cobblerd
[root@linux-node1 ~]# cobbler check
The following are potential configuration items that you may want to fix:
1 : debmirror package is not installed, it will be required to manage debian deployments and repositories
2 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them
Restart cobblerd and then run 'cobbler sync' to apply changes.
</pre>

###1.7 配置DHCP
<pre>
此参数为1时表示使用，cobbler管理dhcp
[root@linux-node1 ~]# sed -i 's#manage_dhcp: 0#manage_dhcp: 1#g' /etc/cobbler/settings
[root@linux-node1 ~]# vim /etc/cobbler/dhcp.template
\#仅列出修改过的部分
\......
subnet 192.168.56.0 netmask 255.255.255.0 {
     option routers             192.168.56.2;
     option domain-name-servers 192.168.56.2;
     option subnet-mask         255.255.255.0;
     range dynamic-bootp        192.168.56.200 192.168.56.254;
\......
</pre>
###1.8 同步Cobbler
<pre>
\#同步最新cobbler配置，它会根据配置自动修改dhcp等服务。
[root@linux-node1 ~]# systemctl restart xinetd 
[root@linux-node1 ~]# systemctl restart cobblerd
[root@linux-node1 ~]# cobbler sync
task started: 2016-06-02_231553_sync
task started (id=Sync, time=Thu Jun  2 23:15:53 2016)
running pre-sync triggers
cleaning trees
removing: /var/lib/tftpboot/grub/images
copying bootloaders
trying hardlink /var/lib/cobbler/loaders/pxelinux.0 -> /var/lib/tftpboot/pxelinux.0
trying hardlink /var/lib/cobbler/loaders/menu.c32 -> /var/lib/tftpboot/menu.c32
trying hardlink /var/lib/cobbler/loaders/yaboot -> /var/lib/tftpboot/yaboot
trying hardlink /usr/share/syslinux/memdisk -> /var/lib/tftpboot/memdisk
trying hardlink /var/lib/cobbler/loaders/grub-x86.efi -> /var/lib/tftpboot/grub/grub-x86.efi
trying hardlink /var/lib/cobbler/loaders/grub-x86_64.efi -> /var/lib/tftpboot/grub/grub-x86_64.efi
copying distros to tftpboot
copying images
generating PXE configuration files
generating PXE menu structure
rendering DHCP files
generating /etc/dhcp/dhcpd.conf
rendering TFTPD files
generating /etc/xinetd.d/tftp
cleaning link caches
running post-sync triggers
running python triggers from /var/lib/cobbler/triggers/sync/post/*
running python trigger cobbler.modules.sync_post_restart_services
running: dhcpd -t -q
received on stdout: 
received on stderr: 
running: service dhcpd restart
received on stdout: 
received on stderr: Redirecting to /bin/systemctl restart  dhcpd.service
running shell triggers from /var/lib/cobbler/triggers/sync/post/*
running python triggers from /var/lib/cobbler/triggers/change/*
running python trigger cobbler.modules.scm_track
running shell triggers from /var/lib/cobbler/triggers/change/*
*** TASK COMPLETE ***

#查看dhcp配置文件的标注开头证明自己由Cobbler管理
[root@linux-node1 ~]# less /etc/dhcp/dhcpd.conf 
\# ******************************************************************
\# Cobbler managed dhcpd.conf file
\# generated from cobbler dhcp.conf template (Thu Jun  2 15:15:54 2016)
\# Do NOT make changes to /etc/dhcpd.conf. Instead, make your changes
\# in /etc/cobbler/dhcp.template, as /etc/dhcpd.conf will be
\# overwritten.
\# ******************************************************************
ddns-update-style interim;
\......
</pre>

###Cobbler的命令行管理
\#查看帮助
<pre>
[root@linux-node1 ~]# cobbler
usage
=====
cobbler <distro|profile|system|repo|image|mgmtclass|package|file> ... 
        [add|edit|copy|getks*|list|remove|rename|report] [options|--help]
cobbler <aclsetup|buildiso|import|list|replicate|report|reposync|sync|validateks|version|signature|get-loaders|hardlink> [options|--help]
</pre>
\#导入镜像参数
<pre>
[root@linux-node1 ~]# cobbler import --help
Usage: cobbler [options]

Options:
  -h, --help            show this help message and exit
  --arch=ARCH           OS architecture being imported
  --breed=BREED         the breed being imported
  --os-version=OS_VERSION
                        the version being imported
  --path=PATH           local path or rsync location
  --name=NAME           name, ex 'RHEL-5'
  --available-as=AVAILABLE_AS
                        tree is here, don't mirror
  --kickstart=KICKSTART_FILE
                        assign this kickstart file
  --rsync-flags=RSYNC_FLAGS
                        pass additional flags to rsync
</pre>
\#常见参数注释：
<pre>
cobbler check    核对当前设置是否有问题
cobbler list     列出所有的cobbler元素
cobbler report   列出元素的详细信息
cobbler sync     同步配置到数据目录,更改配置最好都要执行下
cobbler reposync 同步yum仓库
cobbler distro   查看导入的发行版系统信息
cobbler system   查看添加的系统信息
cobbler profile  查看配置信息

#可单个执行查看帮助信息
例：
[root@linux-node1 ~]# cobbler distro
usage
=====
cobbler distro add
cobbler distro copy
cobbler distro edit
cobbler distro find
cobbler distro list
cobbler distro remove
cobbler distro rename
cobbler distro report
</pre>






后文补充：
cobbler distro remove --name=CentOS-7







