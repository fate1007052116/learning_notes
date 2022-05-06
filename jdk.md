# jdk

## 一、安装jdk

```shell
# jdk12源码包下的 doc/building.html 里面有写
alias yum="yum -y"
sudo yum install freetype-deve
sudo yum install cups-devel
sudo yum install libXtst-devel libXt-devel libXrender-devel libXrandr-devel libXi-devel
sudo yum install alsa-lib-devel
sudo yum install libffi-devel
sudo yum install autoconf
```



### 1.ubuntu编译

> 注意：一定要装一个干净的ubuntu
>
> `jdk`源码下载地址：`http://hg.openjdk.java.net/jdk`
>
> 如果老是失败，可以去`github`上下载源码包`https://github.com/openjdk/jdk/releases/tag/jdk-12-ga`
>
> `https://github.com/openjdk/jdk/releases/tag/jdk-11+27`

```shell
# 如果一次成功不了，就装一个新的ubuntu 20.4 ，使用原生的 apt 源（可能有关系）
luo@ubuntu:~$ sudo apt install -y build-essential

# 安装 build-essential 漏掉的依赖
sudo apt-get install autoconf -y 

sudo apt-get install libx11-dev libxext-dev libxrender-dev libxrandr-dev libxtst-dev libxt-dev -y

sudo apt-get install libcups2-dev -y
 
sudo apt-get install libfontconfig1-dev -y

sudo apt-get install libasound2-dev -y

# 装一个比要编译的jdk版本高的jdk
luo@ubuntu:~$ sudo apt install -y openjdk-14-jdk

# jdk 版本不对
configure: (Your Boot JDK version must be one of: 11 12)

# 找出已安装的jdk
apt list --installed | grep jdk
openjdk-14-jdk/focal-updates,now 14.0.2+12-1~20.04 amd64 [installed]

# 删除
 sudo apt remove openjdk-14-jdk
 
 # 重新安装jdk11
 sudo apt install -y openjdk-11-jdk
 
 # 配置jdk编译参数
luo@ubuntu:~/jvm/jdk12-06222165c35f$ bash configure --enable-debug --with-jvm-variants=server --with-version-pre=java-12-luojunhua

```

```shell
# 配置成功将显示
====================================================
A new configuration has been successfully created in
/home/luo/jvm/jdk12-06222165c35f/build/linux-x86_64-server-fastdebug
using configure arguments '--enable-debug --with-jvm-variants=server --with-version-pre=java-12-luojunhua'.

Configuration summary:
* Debug level:    fastdebug
* HS debug level: fastdebug
* JVM variants:   server
* JVM features:   server: 'aot cds cmsgc compiler1 compiler2 epsilongc g1gc graal jfr jni-check jvmci jvmti management nmt parallelgc serialgc services shenandoahgc vm-structs zgc'
* OpenJDK target: OS: linux, CPU architecture: x86, address length: 64
* Version string: 12-javaluojunhua+0-adhoc.luo.jdk12-06222165c35f (12-javaluojunhua)

Tools summary:
* Boot JDK:       openjdk version "11.0.10" 2021-01-19 OpenJDK Runtime Environment (build 11.0.10+9-Ubuntu-0ubuntu1.20.04) OpenJDK 64-Bit Server VM (build 11.0.10+9-Ubuntu-0ubuntu1.20.04, mixed mode, sharing)  (at /usr/lib/jvm/java-11-openjdk-amd64)
* Toolchain:      gcc (GNU Compiler Collection)
* C Compiler:     Version 9.3.0 (at /usr/bin/gcc)
* C++ Compiler:   Version 9.3.0 (at /usr/bin/g++)

Build performance summary:
* Cores to use:   3
* Memory limit:   10945 MB

The following warnings were produced. Repeated here for convenience:
WARNING: --with-version-pre value has been sanitized from 'java-12-luojunhua' to 'javaluojunhua'

```

