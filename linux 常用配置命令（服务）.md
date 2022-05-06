# linux 常用配置命令（服务）

### 1. 进程管理

* ps -aux
* ps -ef
* pstree

### 2. 临时启动/关闭服务(系统重启后失效)

* systemctl status firewalld
* systemctl start firewalld
* systemctl stop firewalld

### 3. 查看服务名

* 方法一：使用 step  ->系统服务就可以看到(CentOS 7 用不了)

* 方法二：ls /etc/init.d

### 4. chkconfig 指令 配置服务的开机启动

* chkconfig --list  centos 7 使用 systemctl list-unit-files
* systemctl list-dependencies sshd

### 5. top 任务管理器（和ps类似，但可以动态显示）

* 按 u 再输入username 可以筛选出该用户的进程
* 按 k 再 输入需要结束的进程 pid
* 按 N 按照pid进行排序
* 按 P 按照cpu占用率排序
* 按 M 按照内存排序
* Top -d 1 -> 1s 刷新一次
### 6. 监控网络服务的状态netstat

* Netstat -anp
* -an 排序
* -p 哪一个进程在调用
### 7. RPM

![rpm](images/rpm.jpeg)

* rpm -qa 查询已安装的rpm列表(noarch 表示 32位 64位通用)
* Rpm -qi  软件名  查询软件包的安装信息
* rpm -ql  软件名   软件包中的文件安装位置
* Rpm -qf 看该文件属于哪一个软件包
* Rpm -e 删除软件
* rpm -e -nodeps 强制删除，依赖于该软件包的软件将无法运行

#### （1）查询已安装软件的位置

```shell
# 1、通过rpm查找软件的安装包名称。(redis举例)
[root@iZ2ze8vj2il5rawecvg57nZ local]# rpm -qa | grep java
javapackages-filesystem-5.3.0-1.module_el8.0.0+11+5b8c10bd.noarch                                                                 
java-1.8.0-openjdk-1.8.0.272.b10-1.el8_2.x86_64                                                                                   
tzdata-java-2020d-1.el8.noarch                                                                                                    
java-1.8.0-openjdk-headless-1.8.0.272.b10-1.el8_2.x86_64 
# 2、通过rpm 找到安装包的安装路径。

[root@iZ2ze8vj2il5rawecvg57nZ local]# rpm -ql java-1.8.0-openjdk-1.8.0.272.b10-1.el8_2.x86_64                                     
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.272.b10-1.el8_2.x86_64/jre/lib/amd64/libjawt.so
# 省略

cd /usr/lib/jvm/jre-1.8.0-openjdk

export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk
```





rpm -qa | grep redis

rpm -ql 安装包名称

#### （2）查询文件属于哪个rpm包

```shell
#<==有时候会发现系统没有某些文件或者命令，但是又不知道这个文件或命令是属于哪个软件包，这时就可以使用-f参数来查询（在有这个文件的系统上查询）。比如本例查询ifconfig命令属于net-tools软件包。
rpm -qf $(which ifconfig)  
net-tools-1.60-110.el6_2.x86_64
```



### 8. 启动之后不进入图形界面（CentOS7之后）
* centos7+版本 

在图形界面使用 ctrl+alt+F2切换到dos界面  dos界面 ctrl+alt+F2切换回图形界面；
在命令上 输入 init 3 命令 切换到dos界面 输入 init 5命令 切换到图形界面；
如果想系统默认 以某种方式启动 使用systemd创建符号链接指向默认运行级别。

 修改方法为：
* 1.首先删除已经存在的符号链接
rm /etc/systemd/system/default.target
* 2.默认级别转换为3(文本模式)
ln -sf /lib/systemd/system/multi-user.target /etc/systemd/system/default.target
或者默认级别转换为5(图形模式)
ln -sf /lib/systemd/system/graphical.target /etc/systemd/system/default.target

简单有效

```shell
# 检查当前默认设置
[root@centos01 ~]# systemctl get-default
graphical.targe # 表示开机将默认进入图形界面

# 设置为命令模式  
systemctl set-default multi-user.target

# 设置为图形模式
systemctl set-default graphical.target
```



### 8. mac 路由操作

```shell
# 添加路由
sh-3.2# sudo route -n add -net 192.168.20.0 -netmask 255.255.255.0 192.168.137.1
# 添加成功的提示
add net 192.168.20.0: gateway 192.168.137.1

# 删除路由
sh-3.2# route -v delete -net 192.168.20.0

# 查看路由
netstat -r
```





### 9. 设置开机自启

* vi /etc/rc.d/rc.local
* 新增 - /etc/init.d/fdfs_trackerd start

### 10. 配置环境变量

不要随意添加空格，会报错

```shell
export JAVA_HOME=/usr/local/jdk1.8.0_251 \
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar \
PATH=$JAVA_HOME/bin:$PATH 

export MAVEN_HOME=/Volumes/extend/temp/apache-maven-3.6.3
export PATH=$PATH:$MAVEN_HOME/bin
```

删除：unset 变量名

> 如果想要使用java的`jps`命令，还需要安装 `yum install java-1.8.0-openjdk-devel.x86_64`

### 11. 使用iTerm2远程连接linux

```shell
luo@luodeMacBook-Pro ~ % ssh root@192.168.2.10
root@192.168.2.10's password:
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Tue Jun 30 20:16:50 2020 from 192.168.2.1
[root@RabbitMQ_1 ~]#
```

### 12.查看可用内存

```shell
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1806         256        1264           9         284        1389
Swap:          2047           0        2047
```



### 13.配置yum源

阿里云镜像配置教程 https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b111KJgSd

```shell
# 备份yum
mv /etc/yum.repos.d/CentOS-Base.repo /opt/

# 下载阿里的yum
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo

# 有时可能无法解析 https://mirrors.aliyun.com/ 的ip地址，即使设置了DNS=114.114.114.114
# 解决办法
# 先去掉 https:// ping 一下，就能获取到yum源了
ping mirrors.aliyun.com

yum makecache
```

> 非阿里云ECS用户会出现 Couldn't resolve host 'mirrors.cloud.aliyuncs.com' 信息，不影响使用。用户也可自行修改相关配置: eg:
>
> `sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo`



### 14.配置环境变量

#### （1）针对所有用户有效

```shell
vim /etc/profile

# 自定义添加环境变量
export JAVA_HOME=/usr/lib/jvm/jre-1.8.0-openjdk
export NODE_HOME=/opt/nodeJs/node-v14.14.0-linux-x64

export PATH=$PATH:$NODE_HOME/bin

# 立即生效
source /etc/profile 
```

#### （2）仅对当前用户有效

```shell
vim ~/.bashrc
```

#### （3）仅对当前会话有效

```shell
# 直接执行
export PATH=/usr/local/mongodb/bin:$PATH
```

### 15.springboot指定端口的三种方式

第一配置文件中添加server.port=9090

第二在命令行中指定启动端口，比如传入参数一server. port=9000   java -jar bootsample. jar -- server.port=9000

第三传入虚拟机系统属性java - Dserver.port=9000 -jar bootsample.jar



### 16.jmeter压测工具乱码

```shell
vi bin/jmeter.properties
```



```properties
sampleresult.default.encoding=UTF-8
```

### 17.centOS 运维神器

```shell
# 安装cockpit
yum -y install cockpit
# 启用cockpit
systemctl enable --now cockpit.socket #设置开机自启动同时开启服务一条命令就可以搞定

# 查看开机是否启动
systemctl list-unit-files|grep cockpit
cockpit-motd.service                        static   
cockpit.service                             static   
cockpit.socket                              enabled 

# enabled表示开机启动

# 启动cockpit服务
systemctl start cockpit.service

# 查看状态
systemctl status cockpit.service

# 使用cockpit监听9090端口
yum install net-tools
netstat -tulnp | grep 9090

# 如果系统防火墙开启，则需要执行以下操作，添加cockpit服务到防火墙以打开9090端口。
firewall-cmd --add-service=cockpit --permanent 
firewall-cmd --reload 

# 浏览器 https://192.168.50.210:9090/ 即可访问
```

### 18、配置apt源

#### （1）[阿里云镜像源](https://www.cnblogs.com/leeyazhou/p/12976814.html#1780123928)

```shell
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```

#### （2）[清华大学镜像源](https://www.cnblogs.com/leeyazhou/p/12976814.html#3924715791)

```shell
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-security main restricted universe multiverse

# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

#### （3）[网易镜像源](https://www.cnblogs.com/leeyazhou/p/12976814.html#3337143666)

```shell
deb http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ focal-backports main restricted universe multiverse
```

#### （4）[中科大镜像源](https://www.cnblogs.com/leeyazhou/p/12976814.html#1569687821)

```shell
deb https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ focal-proposed main restricted universe multiverse
```

编辑文件完成后，执行如下命令进行更新缓存

```shell
sudo apt update 
sudo apt upgrade
```

### 19.ubuntu安装kde桌面

```shell
sudo apt install  plasma-desktop -y

# 安装 xfce桌面（要重启）
sudo apt install xfce4 -y

```

```shell

# 方式二：在 Ubuntu 上安装 Tasksel
sudo apt install tasksel
# 第二步： 安装 kubuntu-desktop
sudo tasksel install kubuntu-desktop
# 或者
sudo tasksel
# 然后手动选择 kubuntu-desktop 进行安装
```

```shell

# 先升级内核，因为 polybar 新版本的ubuntu包管理器中才有
sudo apt-get upgrade linux-image-generic
# 先安装 polybar 组件 https://github.com/polybar/polybar
sudo apt install -y polybar


git clone https://github.com/Yucklys/polybar-nord-theme ~/.config/polybar/

# 安装仿苹果主题
# https://github.com/ayamir/bspwm-dotfiles

```

### 20、挂载远程linux文件系统到本地磁盘

写这篇文章的主要目的就是提供一步一步的指导，关于如何使用 SSHFS 通过 SSH 挂载远程的 Linux 文件系统或目录。

这篇文章对于那些无论出于什么目的，希望在他们本地的系统中挂载远程的文件系统的用户或者系统管理员有帮助。我们通过 Linux 系统中的一个安装了 SSHFS 客户端进行实际测试，并且成功的挂载了远程的文件系统。

在我们进一步安装之前，让我们了解一下 SSHFS 的相关内容，以及它是如何工作的。

#### **什么是 SSHFS？**

SSHFS（Secure SHell FileSystem）是一个客户端，可以让我们通过 SSH 文件传输协议（SFTP）挂载远程的文件系统并且在本地机器上和远程的目录和文件进行交互。

SFTP 是一种通过 SSH 协议提供文件访问、文件传输和文件管理功能的安全文件传输协议。因为 SSH 在网络中从一台电脑到另一台电脑传输文件的时候使用数据加密通道，并且 SSHFS 内置在 FUSE（用户空间的文件系统）内核模块，允许任何非特权用户在不修改内核代码的情况下创建他们自己的文件系统。

在这篇文章中，我们将会向你展示在任意 Linux 发行版上如何安装并且使用 SSHFS 客户端，在本地 Linux 机器上挂载远程的 Linux 文件系统或者目录。

#### **步骤 1：在 Linux 系统上安装 SSHFS**

默认情况下，sshfs 包不存在所有的主流 Linux 发行版中，你需要在你的 Linux 系统中启用 epel 仓库，在 Yum 命令行的帮助下安装 SSHFS 及其依赖。

```shell
#yum install sshfs
#dnf install sshfs              【在 Fedora 22+ 发行版上】
$sudo apt-get install sshfs     【基于 Debian/Ubuntu 的系统】123
```

#### **步骤 2：创建 SSHFS 挂载目录**

当你安装 SSHFS 包之后，你需要创建一个挂载点目录，在这儿你将要挂载你的远程文件系统。例如，我们在 /mnt/tecmint 下创建挂载目录。

```shell
# mkdir /mnt/tecmint
$ sudo mkdir /mnt/tecmint     【基于 Debian/Ubuntu 的系统】12
```

#### **步骤 3：使用 SSHFS 挂载远程的文件系统**

当你已经创建你的挂载点目录之后，现在使用 root 用户运行下面的命令行，在 /mnt/tecmint 目录下挂载远程的文件系统。视你的情况挂载目录可以是任何目录。

下面的命令行将会在本地的 /mnt/tecmint 目录下挂载一个叫远程的一个 /home/tecmint 目录。（不要忘了使用你的 IP 地址和挂载点替换 x.x.x.x）。

```shell
sshfs tecmint@x.x.x.x:/home/tecmint/ /mnt/tecmint
$ sudo sshfs -o allow_other tecmint@x.x.x.x:/home/tecmint/ /mnt/tecmint      【基于 Debian/Ubuntu 的系统】12


sudo sshfs -o allow_other luo@192.168.50.205:/Users/luo/ /mnt/mac
```

如果你的 Linux 服务器配置为基于 SSH 密钥授权，那么你将需要使用如下所示的命令行指定你的公共密钥的路径。

```shell
# sshfs -o IdentityFile=~/.ssh/id_rsa tecmint@x.x.x.x:/home/tecmint/ /mnt/tecmint
$ sudo sshfs -o allow_other,IdentityFile=~/.ssh/id_rsa tecmint@x.x.x.x:/home/tecmint/ /mnt/tecmint     【基于 Debian/Ubuntu 的系统】12
```

#### **步骤 4：验证远程的文件系统挂载成功**

如果你已经成功的运行了上面的命令并且没有任何错误，你将会看到挂载在 /mnt/tecmint 目录下的远程的文件和目录的列表

```shell
# cd /mnt/tecmint
# ls
[root@ tecmint]# ls
12345.jpg                       ffmpeg-php-0.6.0.tbz2                Linux                                           news-closeup.xsl     s3.jpg
cmslogs                         gmd-latest.sql.tar.bz2               Malware                                         newsletter1.html     sshdallow
epel-release-6-5.noarch.rpm     json-1.2.1                           movies_list.php                                 pollbeta.sql
ffmpeg-php-0.6.0                json-1.2.1.tgz                       my_next_artical_v2.php                          pollbeta.tar.bz21234567
```

#### **步骤 5：使用 df -hT 命令检查挂载点**

如果你运行 df -hT命令，你将会看到远程文件系统的挂载点。

```
# df -hT1
```

样本输出：

```shell
Filesystem                          Type        Size  Used Avail Use% Mounted on
udev                                devtmpfs    730M     0  730M   0% /dev
tmpfs                               tmpfs       150M  4.9M  145M   4% /run
/dev/sda1                           ext4         31G  5.5G   24G  19% /
tmpfs                               tmpfs       749M  216K  748M   1% /dev/shm
tmpfs                               tmpfs       5.0M  4.0K  5.0M   1% /run/lock
tmpfs                               tmpfs       749M     0  749M   0% /sys/fs/cgroup
tmpfs                               tmpfs       150M   44K  150M   1% /run/user/1000
tecmint@192.168.0.102:/home/tecmint fuse.sshfs  324G   55G  253G  18% /mnt/tecmint123456789
```

#### **步骤 6：永久挂载远程文件系统**

为了永久的挂载远程的文件系统，你需要修改一个叫 /etc/fstab 的文件。照着做，使用你最喜欢的编辑器打开文件。

```shell
# vi /etc/fstab
$ sudo vi /etc/fstab     【基于 Debian/Ubuntu 的系统】12
```

移动到文件的底部并且添加下面的一行，保存文件并退出。下面条目表示使用默认的设置挂载远程的文件系统。

```shell
sshfs#tecmint@x.x.x.x:/home/tecmint/ /mnt/tecmint fuse.sshfs defaults 0 01
```

确保服务器之间允许 SSH 无密码登录，这样系统重启之后才能自动挂载文件系统。

如果你的服务器配置为基于 SSH 密钥的认证方式，请加入如下行：

```shell
sshfs#tecmint@x.x.x.x:/home/tecmint/ /mnt/tecmint fuse.sshfs IdentityFile=~/.ssh/id_rsa defaults 0 01
```

接下来，你需要更新 fstab 文件使修改生效。

```shell
# mount -a
$ sudo mount -a   【基于 Debian/Ubuntu 的系统】12
```

#### **步骤 7：卸载远程的文件系统**

```shell
# umount /mnt/tecmin
```

### 21、java没有jps指令

```shell
yum install -y  java-1.8.0-openjdk-devel
```

### 22、docker 安装 oss

```shell
sudo  docker run -d -p 7000:9000  -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE"  -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" -p 10000:10000   minio/minio server /data  --console-address ":10000"
```

### 23、查看进程中的线程

```shell
top -H -p pid
jstack 
```

### 24、下载maven依赖，并放到lib目录

```shell
 mvn -DoutputDirectory=./lib -DgroupId=com.it18zhang -DartifactId=CallLogConsumerModule -Dversion=1.0-SNAPSHOT dependency:copy-dependencies

```



### 25、系统启动时挂载硬盘

```shell
# 查看所有磁盘
[luo@localhost ~]$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
|-sda1   8:1    0  300M  0 part /boot
|-sda2   8:2    0    2G  0 part [SWAP]
`-sda3   8:3    0 17.7G  0 part /
sdb      8:16   0   57G  0 disk
`-sdb1   8:17   0   57G  0 part
sr0     11:0    1 1024M  0 rom

# 查看磁盘的uuid
[luo@localhost ~]$ sudo blkid
/dev/sda1: UUID="a8618d78-5588-4ec4-b50d-af4b29de62e2" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="273bc342-01"
/dev/sda2: UUID="43b701ef-b39a-4c8e-841b-9b6c6f6a0e0c" TYPE="swap" PARTUUID="273bc342-02"
/dev/sda3: UUID="8b8efce5-0d69-415d-854b-920b3620a5f3" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="273bc342-03"
/dev/sdb1: UUID="a9ffe590-9fdd-42a3-b718-6c58c57d8958" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="b53fc189-d664-1e45-8bc5-580f3a585f60"

# 编辑
[luo@localhost ~]$ sudo vi /etc/fstab

UUID=8b8efce5-0d69-415d-854b-920b3620a5f3 /                       xfs     defaults        0 0
UUID=a8618d78-5588-4ec4-b50d-af4b29de62e2 /boot                   xfs     defaults        0 0
UUID=43b701ef-b39a-4c8e-841b-9b6c6f6a0e0c none                    swap    defaults        0 0

# 新挂载的磁盘
UUID=a9ffe590-9fdd-42a3-b718-6c58c57d8958 /mnt            ext4     defaults       0 0
```

### 26、查看磁盘剩余空间

```shell
df -h 

du -h --max-depth=0

# mac 环境
du -h -d 1
比较常用的几个选项：

-d：指定目标文件夹的统计层数，-d 0统计整个文件夹大小，-d 1统计文件夹下第一层的文件大小，以此类推

-h：显示人类可以读懂的单位（K、M、G）

-s：统计单个文件、文件夹的大小，等同于-d 0

不指定file参数，会统计当前文件夹下的所有文件的大小

```

### 27、linux 无法安装 vmware 驱动

手动安装

挂载vmware的驱动到linux

![6E4857BB-9B2A-4003-BF19-27A77BD2F803](./images/6E4857BB-9B2A-4003-BF19-27A77BD2F803.png)

然后将

```shell
 tar -zxf VMwareTools-10.3.21-14772444.tar.gz
 cd vmware-tools-distrib/
 
 # 安装脚本的位置
 ./vmware-install.pl
```



### 28、maven单独打包工程中的某一个模块

```shell
# -pl 指定要打包模块相对于顶级父工程的路径
# -am 可以一起打包目标模块所依赖的模块
mvn -s /Users/luo/maven/apache-maven-3.6.3/conf/settings.xml -DskipTests=true -pl iwmake-modules/iwmake-order install
```

### 29、lsof：查看进程打开的文件

lsof全名为list open files，也就是列举系统中已经被打开的文件，通过lsof命令，就可以根据文件找到对应的进程信息，也可以根据进程信息找到进程打开的文件。

![epub_922339_453](images/epub_922339_453.jpeg)

```shell
# 查看谁占用了 8848 端口
lsof -i :8848                                                                                 1 ↵  5198  09:37:10
COMMAND  PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
java    2812  luo   49u  IPv6 0xc62790bc6063dd1f      0t0  TCP 192.168.1.140:8848->smartbar.com:35990 (ESTABLISHED)
java    2812  luo  111u  IPv6 0xc62790bc60da203f      0t0  TCP *:8848 (LISTEN)
java    2812  luo  118u  IPv6 0xc62790bc57d2637f      0t0  TCP 192.168.1.140:8848->smartbar.com:35486 (ESTABLISHED)
java    2812  luo  119u  IPv6 0xc62790bc717939df      0t0  TCP 192.168.1.140:8848->smartbar.com:56772 (ESTABLISHED)
java    2812  luo  121u  IPv6 0xc62790bc57d269df      0t0  TCP 192.168.1.140:8848->smartbar.com:36092 (ESTABLISHED)
java    2812  luo  122u  IPv6 0xc62790bc68a00d1f      0t0  TCP 192.168.1.140:8848->smartbar.com:56408 (ESTABLISHED)
java    2812  luo  123u  IPv6 0xc62790bc57d2703f      0t0  TCP 192.168.1.140:8848->smartbar.com:36094 (ESTABLISHED)
java    2812  luo  124u  IPv6 0xc62790bc5dd4469f      0t0  TCP 192.168.1.140:8848->smartbar.com:35196 (ESTABLISHED)
java    2812  luo  126u  IPv6 0xc62790bc7171669f      0t0  TCP 192.168.1.140:8848->smartbar.com:35170 (ESTABLISHED)
```

### 30、iftop：动态显示网络接口流量信息

iftop是一款实时流量监控工具，可用于监控TCP/IP连接等，必须以root用户的身份运行。一般最小化安装系统都是没有这个命令的，需要使用yum命令额外安装，而且还要从epel源下载。epel源的安装帮助请参见：http://mirrors.aliyun.com/help/epel。安装iftop命令的步骤请参见：

```shell
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
yum -y install iftop
```

![iftop动态显示网络接口流量信息](images/iftop动态显示网络接口流量信息.jpeg)

### 31、iostat：I/O信息统计

iostat是I/O statistics（输入/输出统计）的缩写，其主要功能是对系统的磁盘I/O操作进行监视。它的输出主要是显示磁盘读写操作的统计信息，同时也会给出CPU的使用情况。同vmstat命令一样，iostat命令也不能对某个进程进行深入分析，仅会对系统的整体情况进行分析

![iostat](images/iostat.jpeg)



```shell
# mac 系统下
iostat -c 100
```



### 32、高级排序

```shell
#     -k field1[,field2], --key=field1[,field2]
             Define a restricted sort key that has the starting position field1, and optional ending position field2 of
             a key field.  The -k option may be specified multiple times, in which case subsequent keys are compared
             when earlier keys compare equal.  The -k option replaces the obsolete options +pos1 and -pos2, but the old
             notation is also supported.

# -t char, --field-separator=char
             Use char as a field separator character.  The initial char is not considered to be part of a field when
             determining key offsets.  Each occurrence of char is significant (for example, ``charchar'' delimits an
             empty field).  If -t is not specified, the default field separator is a sequence of blank space charac-
             ters, and consecutive blank spaces do not delimit an empty field, however, the initial blank space is con-
             sidered part of a field when determining key offsets.  To use NUL as field separator, use -t '\0'.

# sort -t 参数指定分隔符， -k 指定了分割之后的使用哪一个位置上的key进行排序
# sort -g 以数字的方式进行排序
du -g -d 1 | sort -t G -k 1 -g # 单位为 GB

du -m -d 1 | sort -t m -k 1 -g # 单位为 MB 兆
```



### 33、配置nginx为文件服务器

```shell
docker run --name nginx -p 8888:80 -v /Users/luo/Downloads/:/usr/share/nginx/html -v /Users/luo/support_software/nginx/default.conf:/etc/nginx/conf.d/default.conf  nginx

cat /etc/nginx/conf.d/default.conf
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;
    
    charset utf-8;# 注意编码,windows 平台，编码应该为gbk

    location / {
        autoindex on;  # 允许显示网站目录文件列表
        autoindex_exact_size off;
        autoindex_localtime on;
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
}
```



## 二、配置网卡

### 1. 新增网卡之后的配置

```shell
[luo@localhost ~]$ cd /etc/sysconfig/network-scripts/
[luo@localhost network-scripts]$ ls
ifcfg-ens32
# 拷贝一份网卡配置文件
[luo@localhost network-scripts]$ cp ifcfg-ens33 ifcfg-ens32
# 编辑新增的网卡配置文件
[luo@localhost network-scripts]$ vi ifcfg-ens32
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static # 更改为静态ip
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens32						# 设备名称，不能乱改，改了之后nmcli 无法找到该网卡
DEVICE=ens32					# 配置设备
											# 删除网卡的UUID，不删除将导致。如果两张网卡UUID一致，则只能启动一张网卡，另外一张网卡没有ip地址
ONBOOT=yes						# 开机启动
DNS1=8.8.8.8
IPV6_PRIVACY=no
IPADDR=192.168.137.20 # 配置ip地址
											# 这里的网关被我删除了
											# 网卡的mac地址也删了
PREFIX=24

# centOS 8 下直接使得刚配置的网卡配置生效
[root@localhost network-scripts]# nmcli connection reload
[root@localhost network-scripts]# nmcli c up ens33
```

### 2. 显示连接信息

```shell
[root@localhost network-scripts]# nmcli connection
NAME     UUID                                  TYPE      DEVICE
ens33    5c3462cf-0556-46e1-bbe4-caddb644f97f  ethernet  ens33
ens32    152beb06-47c5-c5e8-95a9-385590654382  ethernet  ens32
docker0  67ad87e7-e453-4696-94d8-a961d708ab35  bridge    docker0
virbr0   736b7920-185b-4c0a-bc45-3fb954e4e48e  bridge    virbr0
```

### 3. 显示具体的网络接口信息

```shell
[root@localhost network-scripts]# nmcli connection show ens33
connection.id:                          ens33
connection.uuid:                        5c3462cf-0556-46e1-bbe4-caddb644f97f
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              ens33
connection.autoconnect:                 是
connection.autoconnect-priority:        0
connection.autoconnect-retries:         -1 (default)
connection.multi-connect:               0（default）
# ... 省略
```

### 4. 将linux配置为路由器

启用linux的路由功能

如果不启用，linux职能接收数据包和发送数据包

```shell
# 查看linux中ipv4进程的相关配置
[root@localhost network-scripts]# cd /proc/sys/net/ipv4
[root@localhost ipv4]# ls
cipso_cache_bucket_size            tcp_ecn_fallback
cipso_cache_enable                 tcp_fack
cipso_rbm_optfmt                   tcp_fastopen
cipso_rbm_strictvalid              tcp_fastopen_blackhole_timeout_sec
conf                               tcp_fastopen_key
fib_multipath_hash_policy          tcp_fin_timeout
fib_multipath_use_neigh            tcp_frto
fwmark_reflect                     tcp_fwmark_accept
icmp_echo_ignore_all               tcp_invalid_ratelimit
icmp_echo_ignore_broadcasts        tcp_keepalive_intvl
icmp_errors_use_inbound_ifaddr     tcp_keepalive_probes
icmp_ignore_bogus_error_responses  tcp_keepalive_time
icmp_msgs_burst                    tcp_l3mdev_accept
# 查看路由功能是否启用（1是启用，0为未启用）
[root@localhost ipv4]# cat ip_forward
1
# 如果路由功能未启用，使用这个命令临时启用路由功能，network服务重启，就恢复原样  
[root@localhost ipv4]# echo '1' > /proc/sys/net/ipv4/ip_forward

# 永久设置为开启路由功能（centOS 8 已经默认开启路由功能 ）
[root@localhost etc]# vi /etc/sysctl.conf
net.ipv4.ip_forward = 1 

# 使得配置的路由功能生效
[root@localhost etc]# sysctl -p
```

`ubuntu`启用路由功能

```shell
luo@ubuntu:~$ sudo vi  /etc/sysctl.conf

# 将下面这一行取消注释
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

luo@ubuntu:~$ sudo sysctl -p
net.ipv4.ip_forward = 1
```



添加临时路由

```shell
route add -net 192.168.20.20 netmask 255.255.255.0 gw 192.168.30.1

# 删除临时路由 
route del -net 192.168.20.20

# 路由查看
route
```

添加永久路由

```shell
# 创建该网卡对应的路由文件
[root@localhost network-scripts]# vi /etc/sysconfig/network-scripts/route-ens37 
ADDRESS0=192.168.40.0                                                                                                                          
NETMASK0=255.255.255.0                                                                                                                         
GATEWAY0=192.168.30.20 
# 多条路由依次
ADDRESS1=xxx                                                                                                                         
NETMASK1=xxx                                                                                                                      
GATEWAY1=xxx
# 序号递增
ADDRESS2=xxx                                                                                                                         
NETMASK2=xxx                                                                                                                      
GATEWAY2=xxx
```

关闭路由的防火墙（针对低版本CentOS）

```shell
# 查看路由表
iptables -L
# 删除防火墙规则（临时 ）
iptables -D FORWARD 1
# 保存防火墙配置
/etc/rc.d/init.d/iptables save
```

查看途经的路由器

```shell
pathping 192.168.1.1
```



### 5. linux添加默认路由

```shell
sudo route add default gw 192.168.43.1 ens33

sudo route del default 
```



### 6、linux安装正向代理软件squid

```shell
 sudo apt install -y squid
 
 cd /etc/squid/
 
 sudo vi squid.conf
 
 # 配置允许访问
 # Deny requests to certain unsafe ports
http_access allow all #deny !Safe_ports

# Deny CONNECT to other than secure SSL ports
http_access  allow all #deny CONNECT !SSL_ports

# Only allow cachemgr access from localhost
http_access allow all  #localhost  manager
http_access allow all #localhost  manager
 
 systemctl restart squid
```



```shell
luo@ubuntu:/etc/squid$ systemctl status  squid
● squid.service - Squid Web Proxy Server
     Loaded: loaded (/lib/systemd/system/squid.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-10-19 20:46:07 PDT; 6min ago
       Docs: man:squid(8)
    Process: 79326 ExecStartPre=/usr/sbin/squid --foreground -z (code=exited, status=0/SUCCESS)
    Process: 79329 ExecStart=/usr/sbin/squid -sYC (code=exited, status=0/SUCCESS)
   Main PID: 79330 (squid)
      Tasks: 4 (limit: 5209)
     Memory: 18.7M
     CGroup: /system.slice/squid.service
             ├─79330 /usr/sbin/squid -sYC
             ├─79332 (squid-1) --kid squid-1 -sYC
             ├─79333 (logfile-daemon) /var/log/squid/access.log
             └─79334 (pinger)

Oct 19 20:46:07 ubuntu squid[79332]: Max Swap size: 0 KB
Oct 19 20:46:07 ubuntu squid[79332]: Using Least Load store dir selection
Oct 19 20:46:07 ubuntu squid[79332]: Set Current Directory to /var/spool/squid
Oct 19 20:46:07 ubuntu squid[79332]: Finished loading MIME types and icons.
Oct 19 20:46:07 ubuntu squid[79332]: HTCP Disabled.
Oct 19 20:46:07 ubuntu squid[79332]: Pinger socket opened on FD 14
Oct 19 20:46:07 ubuntu squid[79332]: Squid plugin modules loaded: 0
Oct 19 20:46:07 ubuntu squid[79332]: Adaptation support is off.
Oct 19 20:46:07 ubuntu squid[79332]: Accepting HTTP Socket connections at local=[::]:3128 remote=[::] FD 12 flags=9
# 从日志中可以看到代理的监听端口
```





## 三、IDEA崩溃之后即使重装也闪退

删除`/Users/luo/Library/Application Support/JetBrains/IntelliJIdea2020.2`下的所有文件

## 四、mac

### 1.设置java环境变量

下载JDK安装包：在JDK1.8下载中选择Mac的JDK安装包

安装JDK：这里只要按照安装指引一步一步安装即可

查找JAVA_HOME

打开Mac的终端，检查JDK是否安装成功：java -version

查看java指令所在的目录：which java。

输出结果：/usr/bin/java

显示java指令文件的属性：ls -l /usr/bin/java。

输出结果如下：

lrwxr-xr-x 1 root wheel 74 12 2 06:44 /usr/bin/java -> /System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/java，从输出结果可以知道**/usr/bin/java文件是一个链接文件，实际是指向/System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands/java**文件的。

进入实际指令所在的文件夹：

cd /System/Library/Frameworks/JavaVM.framework/Versions/Current/Commands。但是这个目录并不是JAVA_HOME目录。

然后就是最重点的地方，在这个目录下面有一个mac的JDK特有的java_home指令可以查看JDK的JAVA_HOME目录。

执行指令：./java_home

执行结果如下：

/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home

设置JAVA_HOME

记得切换成root用户(sudo -i)或者给指令添加sudo

临时有效（重启后失效）

编辑.bash_profile文件：vim ~/.bash_profile

添加以下内容：

```shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home

export PATH=𝐽𝐴𝑉𝐴𝐻𝑂𝑀𝐸/𝑏𝑖𝑛:JAVAHOME/bin:PATH
```

使修改的文件生效：source ~/.bash_profile

永久有效

修改文件操作权限：chmod 777 /etc/profile

编辑/ect/profile文件：vim /etc/profile

添加以下内容：

```shell
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home

export PATH=𝐽𝐴𝑉𝐴𝐻𝑂𝑀𝐸/𝑏𝑖𝑛:JAVAHOME/bin:PATH
```

使修改的文件生效：source /etc/profile

#### big sur 不能启动 nacos 的问题

问题描述

```shell
# mac升级到bigsur后，nacos中startup.sh启动失败，日志显示

# nohup: /Library/Internet: No such file or directory
```



> nacos默认使用mac自带java，路径为 /Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home
>
> 以上路径中存在空格
>
> 在mac之前的版本中，对空格的解析是正常的，但是在mac big sur中，对空格的解析不同（还不知道是不是bug）
>
> 解决方案一：
>
> 可以在nacos的startup.sh中，增加自己的java_home路径（不带空格的）
> ————————————————
> 版权声明：本文为CSDN博主「梦幻雪」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/qq_26162069/article/details/110468070



### 2.安装ruby

`https://raw.githubusercontent.com/rvm/rvm/master/binscripts/rvm-installer`下载安装脚本到本地

然后执行

```shell
vi ./install.sh
chmod 777 ./install.sh
./install.sh

# 安装成功
luo@luojunhuas-iMac ruby_install_shell % ruby -version
ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.x86_64-darwin19]
Traceback (most recent call last):
-e:1:in `<main>': undefined local variable or method `rsion' for main:Object (NameError)
luo@luojunhuas-iMac ruby_install_shell % ruby -version
ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.x86_64-darwin19]
Traceback (most recent call last):
-e:1:in `<main>': undefined local variable or method `rsion' for main:Object (NameError)
```

### 3.设置swap分区

```shell
# 禁用
sudo launchctl unload -w /System/Library/LaunchDaemons/com.apple.dynamic_pager.plist
sudo rm /private/var/vm/swapfile*

# 启用
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.dynamic_pager.plist

```

### 4.查看路由表

```shell
netstat -nr
# 添加路由（这个示例为默认路由）
 sudo route -n add -net 0.0.0.0 -netmask 0.0.0.0 192.168.40.20
```



### 5.安装`home brew`

```shell
# 1、要求先安装 git
sudo yum install git -y

# 2、设置 http_proxy
export http_proxy=192.168.50.20:10809

# 3、设置 git 的代理
git config --global http.proxy 'socks5://192.168.50.20:10808'
# 取消设置
git config --global --unset http.proxy


# 执行 home_brew 安装脚本
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

==> Next steps:
- Add Homebrew to your PATH in /home/luo/.bash_profile:
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/luo/.bash_profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
- Run `brew help` to get started
- Further documentation: 
    https://docs.brew.sh
config: -c: line 777: syntax error: unexpected end of file

# 将 brew 添加到环境变量中
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/luo/.bash_profile

eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# 代码高亮
brew install zsh-syntax-highlighting zsh-autosuggestions 

source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh

source /opt/homebrew/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

```



```shell
# 离线手动安装
git clone https://github.com/Homebrew/brew ~/.linuxbrew/Homebrew
mkdir ~/.linuxbrew/bin
ln -s ~/.linuxbrew/Homebrew/bin/brew ~/.linuxbrew/bin
eval $(~/.linuxbrew/bin/brew shellenv)
```

#### （1）设置 homebrew 的镜像源

> https://mirrors.ustc.edu.cn/help/brew.git.html
>
> https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/
>
> https://docs.brew.sh/Taps

替换 USTC 镜像：

```shell
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
```

重置为官方地址：

```shell
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git
```

#### （2）安装`oh my zsh`

> onMyZsh gitHub 地址（安装教程）：https://github.com/ohmyzsh/ohmyzsh
>
> 字体地址：https://github.com/powerline/fonts

```shell
# 如果是 centOS 需要先安装zsh
sudo yum install -y zsh

# 建议先配置一下 git 代理
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 设置主题
vi ~/.zshrc
ZSH_THEME="agnoster"

# 此主题需要安装字体
git clone https://github.com/powerline/fonts.git --depth=1

# 安装字体
cd fonts
./install.sh

# 如果要在 iterm 上使用，item2 > preference->profile->text 需要将字体设置为 inconsolata for powerline
# 这个字体在上面安装完字体之后会自动搜索得到
```

#### （3）安装 oh my zsh 插件

```shell
# 语法高亮插件 zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting

# 自动补全插件 zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions

# 自动跳转插件 autojump
git clone git://github.com/joelthelion/autojump.git
# 进入clone目录，接着执行安装文件
cd autojump
./install.py
# 接着根据安装完成后的提示，在~/.bashrc最后添加下面语句：
vim ~/.bashrc    
[[ -s /home/luo/.autojump/etc/profile.d/autojump.sh ]] && source /home/luo/.autojump/etc/profile.d/autojump.sh

# 安装完后在 zsh 配置文件中启用插件
# 编辑~/.zshrc   
vim ~/.zshrc    
# 在plugins后括号里添加安装的插件名字
plugins=( git 
            autojump 
            zsh-autosuggestions 
            zsh-syntax-highlighting
            )
# 最后刷新
source ~/.zshrc    
```

> autojump 工作原理：它会在你每次启动命令时记录你当前位置，并把它添加进它自身的数据库中。这样，某些目录比其它一些目录添加的次数多，这些目录一般就代表你最重要的目录，而它们的“权重”也会增大。
>
> ```shell
> 查看版本：
> j -v
> 目录跳转：
>  j    [目录的名字或名字的一部分] // 不受当前所在目录的限制
> 
> 查看当前权重：
> j --stat
> 
> 进入权重最高的目录：
> j
> 
> 改变当前目录权重值：
> j -i [权重] // 增加
> j -d [权重] // 减少
> ```
>
> 

### 6. Mac 自带的截屏使用

Command + Shift + 3    截取整个屏幕，保存图片在桌面  
Command + Shift + 4    选取部分屏幕区域，保存图片在桌面 
先 Command + Shift + 4  再空格，可以对指定的窗口或者菜单截屏

以上快捷键，加上 Ctrl，可以把截屏保存在剪贴板。

### 7. MAC查看端口占用情况

命令 lsof -i tcp:port  （port替换成端口号，比如6379）可以查看该端口被什么程序占用，并显示PID，方便KILL

### 8.idea idea.vmoptions 文件位置

```shell
/Users/username/Library/Application Support/JetBrains/IntelliJIdea2020.2/idea.vmoptions
```

### 9. 设置 top 命令按照内存占用来排序

top命令可以看一些负载情况

```shell
top -s 20
```

-s代表每隔20s更新一次，这样可以避免更新太快看不清

第3列%CPU是cpu占用，第8列MEM是内存

按内存排序

```shell
# 在 top 命令里面
orsize #回车

```

按cpu排序
ocpu 回车

就是先按o 会显示
primary key [-pid]:

在后面输入cpu就是按cpu排序，rsize就是内存



### 11、将macOS系统镜像转为iso格式供kvm使用



### 12、vmware fusion 中，win7安装补丁以启用vmware tools

> https://kb.vmware.com/s/article/78708
>
> win7需要下载和安装下面两个补丁
>
> 注意：只有 win7 sp1 才能安装下面的两个补丁
>
> https://www.catalog.update.microsoft.com/search.aspx?q=kb4474419
>
> https://www.catalog.update.microsoft.com/search.aspx?q=4490628



安装VMWare Tools
虽然Win7已经跑起来了，但是分辨率啊，适配啊，什么的看起来都比较别扭，而且还无法使用共享文件夹，所以下一步需要安装VMWare Tools
由于VMWare Tools的一部分组件需要用到SHA-2加密的有关功能，而原始版本的Win7并没有提供，所以需要手动打上补丁，详细问题解析可以看官方文档：https://kb.vmware.com/s/article/78708
下面是具体的步骤：

将补丁拷贝至虚拟机子系统
在虚拟机内打开上文中的百度网盘链接，然后下载那两个以.msu为结尾的补丁（或者你在别的地方下载完了，通过U盘的形式拷进虚拟机也可以）

原文链接：https://blog.csdn.net/weixin_44072270/article/details/118603679

然后在`vmware fusion`的程序文件夹中找到`vmware tools`的安装文件`/Applications/VMware\ Fusion.app/Contents/Library/isoimages/windows.iso`

将其上传至`win7`就能进行相应的安装了

## 五、安卓虚拟机

> 下载地址：https://www.android-x86.org/
>
> 安装简易教程：https://zhuanlan.zhihu.com/p/44782864
>
> 前面的都一样，就是没出现那个Android界面，而是黑屏
>
> 虚拟机设置，显示，显卡控制器改为VBoxVGA
