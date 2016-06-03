# Cobbler 自动化系统部署
自动化操作系统部署Cobbler

>注：本文内容参考自互联网，版权归属于互联网，感谢老男孩、赵班长、徐布斯   
##1.1 Cobbler简介
&emsp;&emsp;Cobbler 可以用来快速建立 Linux 网络安装环境，它已将 Linux 网络安装的技术门槛，从大专以上文化水平，成功降低到初中以下，连补鞋匠都能学会。

&emsp;&emsp;Cobbler是一个Linux服务器安装的服务，可以通过网络启动(PXE)的方式来快速安装、重装物理服务器和虚拟机，同时还可以管理DHCP，DNS等。Cobbler可以使用命令行方式管理，也提供了基于Web的界面管理工具(cobbler-web)，还提供了API接口，可以方便二次开发使用。
#####摘自百度百科
&emsp;&emsp;网络安装服务器套件 Cobbler(补鞋匠)从前，我们一直在做装机民工这份很有前途的职业。自打若干年前 Red Hat 推出了 Kickstart，此后我们顿觉身价倍增。不再需要刻了光盘一台一台地安装 Linux，只要搞定 PXE、DHCP、TFTP，还有那满屏眼花缭乱不知所云的 Kickstart 脚本，我们就可以像哈里波特一样，轻点魔棒，瞬间安装上百台服务器。这一堆花里胡哨的东西可不是一般人都能整明白的，没有大专以上学历，通不过英语四级， 根本别想玩转。总而言之，这是一份多么有前途，多么有技术含量的工作啊。很不幸，Red Hat 最新（Cobbler项目最初在2008年左右发布）发布了网络安装服务器套件 Cobbler(补鞋匠)，它已将 Linux 网络安装的技术门槛，从大专以上文化水平，成功降低到初中以下，连补鞋匠都能学会。对于我们这些在装机领域浸淫多年，经验丰富，老骥伏枥，志在千里的民工兄弟们来说，不为一个晴天霹雳。
##1.2 环境准备
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
##1.3 安装Cobbler
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
##1.4 重要配置文件注释：
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
##1.5 启动服务
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
##1.6 检查Cobbler的配置，如果看不到下面的结果，再次执行systemctl start cobblerd
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
7. 创建默认系统用户及密码：
[root@linux-node1 ~]# openssl passwd -1 -salt 'root' 'password'
$1$root$1fvaXuILgb4rdRlHdQ80N/
修改密码为创建的加密码：
[root@linux-node1 ~]# grep default_password /etc/cobbler/settings
default_password_crypted: "$1$root$1fvaXuILgb4rdRlHdQ80N/"
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
##1.7 配置DHCP
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
##1.8 同步Cobbler配置
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

##1.9Cobbler的命令行管理
* 查看帮助
<pre>
[root@linux-node1 ~]# cobbler
usage
=====
cobbler <distro|profile|system|repo|image|mgmtclass|package|file> ... 
        [add|edit|copy|getks*|list|remove|rename|report] [options|--help]
cobbler <aclsetup|buildiso|import|list|replicate|report|reposync|sync|validateks|version|signature|get-loaders|hardlink> [options|--help]
</pre>
* 导入镜像参数
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
* 常见参数注释：
<pre>
cobbler check    核对当前设置是否有问题
cobbler list     列出所有的cobbler元素
cobbler report   列出元素的详细信息
cobbler sync     同步配置到数据目录,更改配置最好都要执行下
cobbler reposync 同步yum仓库
cobbler distro   查看导入的发行版系统信息
cobbler system   查看添加的系统信息
cobbler profile  查看配置信息
</pre>
* 可单个执行查看帮助信息    
例：
<pre>
[root@linux-node1 ~]# cobbler distro
usage
cobbler distro add
cobbler distro copy
cobbler distro edit
cobbler distro find
cobbler distro list
cobbler distro remove
cobbler distro rename
cobbler distro report
</pre>

##1.10导入镜像文件
* 挂载系统光盘
<pre>
[root@linux-node1 ~]# mount /dev/cdrom /mnt
mount: /dev/sr0 写保护，将以只读方式挂载
</pre>
* 导入操作系统来自于/mnt下
<pre>
[root@linux-node1 ~]# cobbler import --path=/mnt/ --name=CentOS-7.2-x86_64 --arch=x86_64 
task started: 2016-06-03_102402_import
task started (id=Media import, time=Fri Jun  3 10:24:02 2016)
Found a candidate signature: breed=redhat, version=rhel6
Found a candidate signature: breed=redhat, version=rhel7
Found a matching signature: breed=redhat, version=rhel7
Adding distros from path /var/www/cobbler/ks_mirror/CentOS-7.2-x86_64:
creating new distro: CentOS-7.2-x86_64
trying symlink: /var/www/cobbler/ks_mirror/CentOS-7.2-x86_64 -> /var/www/cobbler/links/CentOS-7.2-x86_64
creating new profile: CentOS-7.2-x86_64
associating repos
checking for rsync repo(s)
checking for rhn repo(s)
checking for yum repo(s)
starting descent into /var/www/cobbler/ks_mirror/CentOS-7.2-x86_64 for CentOS-7.2-x86_64
processing repo at : /var/www/cobbler/ks_mirror/CentOS-7.2-x86_64
need to process repo/comps: /var/www/cobbler/ks_mirror/CentOS-7.2-x86_64
looking for /var/www/cobbler/ks_mirror/CentOS-7.2-x86_64/repodata/*comps*.xml
Keeping repodata as-is :/var/www/cobbler/ks_mirror/CentOS-7.2-x86_64/repodata
*** TASK COMPLETE ***
#参数注释：
# --path 镜像路径
# --name 为安装源定义一个名字
# --arch 指定安装源是32位、64位、ia64, 目前支持的选项有: x86│x86_64│ia64
# 安装源的唯一标示就是根据name参数来定义，本例导入成功后，安装源的唯一标示就是：CentOS-7.2-x86_64，如果重复，系统会提示导入失败。
#镜像存放目录，cobbler会将镜像中的所有安装文件拷贝到本地一份，放在/var/www/cobbler/ks_mirror下的CentOS-7.2-x86_64-distro-x86_64目录下。因此/var/www/cobbler目录必须具有足够容纳安装文件的空间。
</pre>
* 区分多样化，在导入一份CentOS-6.6操作系统
<pre>
[root@linux-node1 ~]# cobbler import --path=/mnt/ --name=CentOS-6.6-x86_64 --arch=x86_64
</pre>
* 查看镜像列表：
<pre>
[root@linux-node1 ~]# cobbler distro list
   CentOS-6.6-x86_64
   CentOS-7.2-x86_64
</pre>
* 镜像文件所在位置：
<pre>
[root@linux-node1 ~]# ls /var/www/cobbler/ks_mirror/
CentOS-6.6-x86_64  CentOS-7.2-x86_64  config
</pre>

* 删除指定的镜像文件img
<pre>
[root@linux-node1 ~]# cobbler distro remove --name=CentOS-7xxx
</pre>
##1.11指定ks.cfg文件调整内核参数
&emsp;&emsp;Cobbler使用ks.cfg文件来制定所需要的安装配置，分区，网络，主机名等开机优化操作，还可以指定系统安装相应的软件。   
Cobbler-CentOS-7.2-x86_64.cfg   
Cobbler-CentOS-6.6-x86_64.cfg   
配置文件上传至至附件，见文章底部；

* Cobbler的默认ks.cfg文件存放位置；
<pre>
[root@linux-node1 ~]# ls /var/lib/cobbler/kickstarts/
default.ks    esxi5-ks.cfg      legacy.ks     sample_autoyast.xml  sample_esx4.ks   sample_esxi5.ks  sample_old.seed
esxi4-ks.cfg  install_profiles  pxerescue.ks  sample_end.ks（默认使用的ks文件）        sample_esxi4.ks  sample.ks  
</pre>
* 上传准备好的cfg 文件到/var/lib/cobbler/kickstarts/路径，并修改distro list镜像对应的cfg文件；
<pre>
[root@linux-node1 kickstarts]# cobbler profile edit --name=CentOS-7.2-x86_64 --kickstart=/var/lib/cobbler/kickstarts/CentOS-7.2-x86_64.cfg 
[root@linux-node1 kickstarts]# cobbler profile edit --name=CentOS-6.6-x86_64 --kickstart=/var/lib/cobbler/kickstarts/CentOS-6.6-x86_64.cfg 
</pre>

* CentOS7系统网卡名变成eno…这种，为了运维标准化，我们需要修改为我们常用的eth0，使用下面的参数。但要注意是CentOS7才需要下面的步骤，CentOS6不需要。
<pre>
[root@linux-node1 kickstarts]# cobbler profile edit --name=CentOS-7.2-x86_64 --kopts='net.ifnames=0 biosdevname=0'
</pre>

* 查看安装img镜像文件信息
<pre>
[root@linux-node1 kickstarts]#cobbler distro report
NAME                           : centos-6.6-x86_64		###名称
Architecture                   : x86_64
TFTP Boot Files                : {}
Breed                          : redhat
Comment                        : 
Fetchable Files                : {}
Initrd                         : /var/www/cobbler/ks_mirror/CentOS-6.6-x86_64/images/pxeboot/initrd.img   ###镜像文件
Kernel                         : /var/www/cobbler/ks_mirror/CentOS-6.6-x86_64/images/pxeboot/vmlinuz
Kernel Options                 : {}
Kernel Options (Post Install)  : {}
Kickstart Metadata             : {'tree': 'http://@@http_server@@/cblr/links/CentOS-6.6-x86_64'}
Management Classes             : []
OS Version                     : rhel6
Owners                         : ['admin']
Red Hat Management Key         : <<inherit>>
Red Hat Management Server      : <<inherit>>
Template Files                 : {}
Name                           : CentOS-7.2-x86_64		###名称
Architecture                   : x86_64
TFTP Boot Files                : {}
Breed                          : redhat
Comment                        : 
Fetchable Files                : {}
Initrd                         : /var/www/cobbler/ks_mirror/CentOS-7.2-x86_64/images/pxeboot/initrd.img   ###镜像文件
Kernel                         : /var/www/cobbler/ks_mirror/CentOS-7.2-x86_64/images/pxeboot/vmlinuz
Kernel Options                 : {}
Kernel Options (Post Install)  : {}
Kickstart Metadata             : {'tree': 'http://@@http_server@@/cblr/links/CentOS-7.2-x86_64'}
Management Classes             : []
OS Version                     : rhel7
Owners                         : ['admin']
Red Hat Management Key         : <<inherit>>
Red Hat Management Server      : <<inherit>>
Template Files                 : {}
</pre>
* 查看指定的img镜像文件：
<pre>
[root@linux-node1 kickstarts]# cobbler distro report --name=CentOS-7.2-x86_64
</pre>
* 查看所有的cfg配置文件内容：
<pre>
[root@linux-node1 kickstarts]# cobbler profile report
Name                           : CentOS-6.6-x86_64		###名称
TFTP Boot Files                : {}
Comment                        : 
DHCP Tag                       : default
Distribution                   : CentOS-6.6-x86_64
Enable gPXE?                   : 0
Enable PXE Menu?               : 1
Fetchable Files                : {}
Kernel Options                 : {}
Kernel Options (Post Install)  : {}
Kickstart                      : /var/lib/cobbler/kickstarts/CentOS-6.6-x86_64.cfg		###cfg配置文件
Kickstart Metadata             : {}
Management Classes             : []
Management Parameters          : <<inherit>>
Name Servers                   : []
Name Servers Search Path       : []
Owners                         : ['admin']
Parent Profile                 : 
Internal proxy                 : 
Red Hat Management Key         : <<inherit>>
Red Hat Management Server      : <<inherit>>
Repos                          : []
Server Override                : <<inherit>>
Template Files                 : {}
Virt Auto Boot                 : 1
Virt Bridge                    : xenbr0
Virt CPUs                      : 1
Virt Disk Driver Type          : raw
Virt File Size(GB)             : 5
Virt Path                      : 
Virt RAM (MB)                  : 512
Virt Type                      : kvm   
Name                           : CentOS-7.2-x86_64		###名称
TFTP Boot Files                : {}
Comment                        : 
DHCP Tag                       : default
Distribution                   : CentOS-7.2-x86_64
Enable gPXE?                   : 0
Enable PXE Menu?               : 1
Fetchable Files                : {}
Kernel Options                 : {'biosdevname': '0', 'net.ifnames': '0'}		###修改网卡配置eth0...eth1显示方式；
Kernel Options (Post Install)  : {}
Kickstart                      : /var/lib/cobbler/kickstarts/CentOS-7.2-x86_64.cfg		###cfg配置文件
Kickstart Metadata             : {}
Management Classes             : []
Management Parameters          : <<inherit>>
Name Servers                   : []
Name Servers Search Path       : []
Owners                         : ['admin']
Parent Profile                 : 
Internal proxy                 : 
Red Hat Management Key         : <<inherit>>
Red Hat Management Server      : <<inherit>>
Repos                          : []
Server Override                : <<inherit>>
Template Files                 : {}
Virt Auto Boot                 : 1
Virt Bridge                    : xenbr0
Virt CPUs                      : 1
Virt Disk Driver Type          : raw
Virt File Size(GB)             : 5
Virt Path                      : 
Virt RAM (MB)                  : 512
Virt Type                      : kvm
</pre>
* 可以指定名称查看某一个配置的cfg文件：
<pre>
[root@linux-node1 kickstarts]# cobbler profile report --name=CentOS-7.2-x86_64
</pre>
* 每次修改完都要执行一次同步
<pre>
[root@linux-node1 kickstarts]# cobbler sync
</pre>

##1.12部署操作系统
* 新建一台机器，通过网络启动即可。    
![图-1加载DHCP池选择加载系统类型](https://github.com/baolin2200/Cobbler/blob/master/Image/%E5%9B%BE1-%E9%80%89%E6%8B%A9%E7%B3%BB%E7%BB%9F.jpg)
* 选择从Centos7.2安装系统。   
![图-2选择Centos7.2安装系统](https://github.com/baolin2200/Cobbler/blob/master/Image/%E5%9B%BE2-%E9%80%89%E6%8B%A9Centos7%E7%B3%BB%E7%BB%9F.jpg)
* 登陆操作系统检查配置    
![图-3检查配置](https://github.com/baolin2200/Cobbler/blob/master/Image/%E5%9B%BE3-%E7%99%BB%E9%99%86%E7%B3%BB%E7%BB%9F%E6%9F%A5%E7%9C%8B%E9%85%8D%E7%BD%AE1.jpg)

##1.13定制化安装
1. Cobbler支持设备的物理MAC地址来区分设备，针对不同设备安装操作系统
2. 查看虚拟机的MAC地址方法    
![图-4查看虚拟机设备的MAC地址](https://github.com/baolin2200/Cobbler/blob/master/Image/%E5%9B%BE4-%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%94%9F%E6%88%90MAC%E5%9C%B0%E5%9D%80.jpg)

3. Cobbler可以自定义配置网络接口，通过system来固定机器的IP、掩码、网关、DNS、主机名、等等实现基础环境标准化  
<pre>
[root@linux-node1 ~]# cobbler system add --name=linux-Centos7.2-test --mac=00:50:56:38:F3:C5 --profile=CentOS-7.2-x86_64 \
--ip-address=192.168.56.13 --subnet=255.255.255.0 --gateway=192.168.56.2 --interface=eth0 \
--static=1 --hostname=linux-node2.com --name-servers="192.168.56.2" \
--kickstart=/var/lib/cobbler/kickstarts/CentOS-7.2-x86_64.cfg
\#--name 自定义，但不能重复
</pre>
* 查看定义的列表
<pre>
[root@linux-node1 ~]# cobbler system list
   linux-Centos7.2-test
</pre>
* 打开MAC地址为"00:50:56:38:F3:C5"的机器发现会自动安装预选的操作系统   
![图5-自动安装无需选择](https://github.com/baolin2200/Cobbler/blob/master/Image/%E5%9B%BE5-%E8%87%AA%E5%8A%A8%E5%AE%89%E8%A3%85%E6%97%A0%E9%9C%80%E9%80%89%E6%8B%A9.jpg)
* 检查新安装的设备，IP、掩码、网关、DNS、主机名、等等配置   
![图6-检查自定义配置](https://github.com/baolin2200/Cobbler/blob/master/Image/%E5%9B%BE6-%E6%A3%80%E6%9F%A5%E8%87%AA%E5%AE%9A%E4%B9%89%E9%85%8D%E7%BD%AE.jpg)

##1.14指定设备系统重装
* 有些需要重新安装的系统希望reboot后自动重装
* **在需要重装的机器上面！！！安装koan注意yum源**
<pre>
[root@linux-node2 ~]# rpm -ivh http://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
[root@linux-node2 ~]# yum install -y koan
</pre>
* 检查在Cobbler上面有可选择的img
<pre>
[root@linux-node2 ~]# koan --server=192.168.56.11 --list=profiles
\- looking for Cobbler at http://192.168.56.11:80/cobbler_api
CentOS-6.6-x86_64
CentOS-7.2-x86_64
</pre>
* 指定要选择重装的系统
<pre>
[root@linux-node2 ~]# koan --replace-self --server=192.168.56.11 --profile=CentOS-6.6-x86_64
\- looking for Cobbler at http://192.168.56.11:80/cobbler_api
\- reading URL: http://192.168.56.11/cblr/svc/op/ks/profile/CentOS-6.6-x86_64
install_tree: http://192.168.56.11/cblr/links/CentOS-6.6-x86_64
downloading initrd initrd.img to /boot/initrd.img_koan
url=http://192.168.56.11/cobbler/images/CentOS-6.6-x86_64/initrd.img
\- reading URL: http://192.168.56.11/cobbler/images/CentOS-6.6-x86_64/initrd.img
downloading kernel vmlinuz to /boot/vmlinuz_koan
url=http://192.168.56.11/cobbler/images/CentOS-6.6-x86_64/vmlinuz
\- reading URL: http://192.168.56.11/cobbler/images/CentOS-6.6-x86_64/vmlinuz
\- ['/sbin/grubby', '--add-kernel', '/boot/vmlinuz_koan', '--initrd', '/boot/initrd.img_koan', '--args', '"ks=http://192.168.56.11/cblr/svc/op/ks/profile/CentOS-6.6-x86_64 ksdevice=link kssendmac lang= text "', '--copy-default', '--make-default', '--title=kick1464941350']
\- ['/sbin/grubby', '--update-kernel', '/boot/vmlinuz_koan', '--remove-args=root']
\- reboot to apply changes
</pre>
* 重启需要重装的系统,发现系统已经按照指定的镜像重新安装为CentOS6.6操作系统
<pre>
[root@linux-node2 ~]# reboot
</pre>

##1.15自定义登陆界面
* 打个广告：
<pre>
[root@linux-node1 ~]# grep baolin2200 /etc/cobbler/pxe/pxedefault.template 
MENU TITLE Cobbler | //https://github.com/baolin2200
</pre>
* 每次修改完都要执行一次同步
<pre>
[root@linux-node1 ~]# cobbler sync
</pre>
* 效果图   
![图7-修改登录界面](https://github.com/baolin2200/Cobbler/blob/master/Image/%E5%9B%BE7-%E6%95%88%E6%9E%9C%E6%BC%94%E7%A4%BA.jpg)


##1.16附件
* 请选择右键目标另存为。     
[CentOS-6-x86_64.cfg](https://github.com/baolin2200/Cobbler/blob/master/cfg/CentOS-6-x86_64.cfg)  
[CentOS-7-x86_64.cfg](https://github.com/baolin2200/Cobbler/blob/master/cfg/CentOS-7-x86_64.cfg)