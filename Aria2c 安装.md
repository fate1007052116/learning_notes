# Aria2c 安装

参考教程 https://blog.haitianhome.com/install-aria2-web-ariang.html

### 1. 下载地址

https://github.com/aria2/aria2/tree/release-1.35.0

### 2. 运行

```shell
# 下载 QQ
aria2c  https://dldir1.qq.com/qqfile/QQforMac/QQCatalyst/QQ_8.3.6.2823_beta.dmg

# 查看安装目录
luo@luodeMacBook-Pro aria2 % type aria2c
aria2c is /usr/local/aria2/bin/aria2c
```

### 3. 下载图形界面

https://github.com/mayswind/AriaNg/releases

解压后为html文件，直接双击运行即可

### 4. 创建并修改配置文件

`aria2.conf`

```shell
#文件保存路径设置，请手动更改
dir=/Volumes/extend/temp/aria-download

disk-cache=32M
file-allocation=none
continue=true
max-concurrent-downloads=10
max-connection-per-server=5
min-split-size=10M
split=20
disable-ipv6=true
input-file=/usr/local/aria2/aria2.session
save-session=/usr/local/aria2/aria2.session

## RPC相关设置 ##
# 启用RPC, 默认:false
enable-rpc=true
# 允许所有来源, 默认:false
rpc-allow-origin-all=true
# 允许非外部访问, 默认:false
rpc-listen-all=true
# 事件轮询方式, 取值:[epoll, kqueue, port, poll, select], 不同系统默认值不同
#event-poll=select
# RPC监听端口, 端口被占用时可以修改, 默认:6800
rpc-listen-port=6800
# 设置的RPC授权令牌，在设置AriaNg时需要用到，请手动更改
# 请使用数字密码，英文密码会出问题
rpc-secret=235689

follow-torrent=true
listen-port=6881-6999
enable-dht=true
enable-peer-exchange=true
peer-id-prefix=-TR2770-
user-agent=Transmission/2.77
seed-ratio=0.1
bt-seed-unverified=true
bt-save-metadata=false

```

### 5. 创建文件

```shell
mkdir /usr/local/aria2
touch /usr/local/aria2/aria2.session
sudo chmod 777 /usr/local/aria2/aria2.session
```

### 6. 启动

```shell
luo@luodeMacBook-Pro aria2 % pwd
/usr/local/aria2
# -D 代表守护进程的形式启动
luo@luodeMacBook-Pro aria2 % ./bin/aria2c aria2c --conf-path=./conf/aria2.conf -D
```

### 7. 设置密码

密码是`aria2.conf`中的`rpc-secret=235689`

![Snip20200814_10](/Users/luo/Documents/开发笔记/images/Snip20200814_10.png)