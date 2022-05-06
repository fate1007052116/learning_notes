# FastDfs 官方安装教程

https://github.com/happyfish100/fastdfs/blob/fde110996b477183b49eb5da55e3775a99719f65/INSTALL

### 因为fast_dfs 是用c开发的，需要编译安装，需要安装
* yum install -y make
* yum install -y cmake
* Yum install -y gcc
* Yum install -y gcc-c++

### 安装fast_dfs 之前 需要安装依赖库 fastdfs100-libfastcommon-master.zip

* cd  libfastcommon
* ./make.sh clean && ./make.sh && ./make.sh install  借助 libfastcommon 提供的make.sh 进行安装
### 以相同的方式安装fast_dfs 本体
* cd fastdfs/

* ./make.sh clean && ./make.sh && ./make.sh install

### 初始化配置

* cd fastdfs/
* ./setup.sh /etc/fdfs
  
### 将fast_dfs 配置成tracker,storage,client

* vi /etc/fdfs/tracker.conf   
  * port = 22122 配置tracker监听端口
  * bind_addr =  可以限定只允许该ip访问tracker （默认不填）
  * base_path = /home/fastdfs_tracker 配置数据的根路径，fast_dfs 并不会帮我们创建这个目录，目录需要手动创建
* vi /etc/fdfs/storage.conf    
  * base_path = /home/fastdfs_storage_base_path 配置数据的根路径，fast_dfs 并不会帮我们创建这个目录，目录需要手动创建
    * storage启动成功之后，在base_path/logs/storaged.log 存放启动日志（连接tracker是否成功可以在这里查看）
    * base_path/data/fdfs_storaged.pid 查看进程pid
    * base_path/data/storage_stat.dat 查看统计信息
    * base_path/data/sync 查看与其他storage 的同步信息
  * store_path0 = /home/fastdfs_store_true_data/ 更改存储路径，一台服务器可以配置多个文件存储路径
    * store_path0/data/有256个子目录（16进制的文件夹名字），每一个子目录下还有256个子目录，总共 （256*256）用来真正的存储用户的数据
  * tracker_server = 192.168.209.121:22122    配置tracker 所在ip+端口号
* vi /etc/fdfs/client.conf
  * base_path = /home/client
  * tracker_server = 192.168.2.10:22122
### 启动fast_dfs

* fast_dfs 的 服务(service) 会和mysql一样注册到 /etc/init.d/ 目录下
* 可以直接使用systemctl start fdfs_trackerd 启动 tracker 服务
* 或者 /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
* systemctl status fdfs_trackerd 可以查看服务启动状态
* systemctl stop fdfs_trackerd 停止服务
* tracker 启动之后，storage才能启动 

### 通过fdfs_upload_file上传文件(写操作必须经过tracker的中转，删除也是写操作)

#### fdfs_upload_file /etc/fdfs/client.conf apache-zookeeper-3.5.8-bin.tar.gz

* fdfs_upload_file client.conf配置文件   要上传的文件

* 全写/usr/bin/fdfs_upload_file /etc/fdfs/client.conf apache-zookeeper-3.5.8-bin.tar.gz

* 上传成功之后，会返回一个

  * group1/M00/00/00/wKgCFF7wLKOAWKlWAI9aDCV_Fz0.tar.gz （删除和读的时候通过这个标识）

  * 卷名/虚拟名/16进制目录/16进制目录/文件名

  * 真实的文件就存储在storage group1的 /home/fast_dfs_data/data/00/00/wKgCFF7wLKOAWKlWAI9aDCV_Fz0.tar.gz

### 删除文件

* fdfs_delete_file /etc/fdfs/client.conf group1/M00/00/00/wKgCFF7wLKOAWKlWAI9aDCV_Fz0.tar.gz

* 删除文件需要找到tracker



# Nginx + fast_dfs

需要fastdfs-nginx-module模块的支持

**[happyfish100](https://gitee.com/fastdfs100) / [fastdfs-nginx-module](https://gitee.com/fastdfs100/fastdfs-nginx-module)**

*  fastdfs-nginx-module 依赖于 fast_fdfs 组件 以及 libfastdfscommon ， 所以需要通过config来配置两者的位置

* 如果在安装fast_dfs 的时候 ./make.sh 编译安装的时候，fast_dfs 的安装目录是在 /usr/bin/ 以外的目录，需要修改fastdfs-nginx-module/src/config 文件 中的 CORE_INCS="$CORE_INCS /usr/local/include" 为
CORE_INCS="$CORE_INCS /usr/local/include/fastcommon /usr/local/include/fastdfs"

* fast_dfs 默认安装的时候也需要修改为

  ```shell
  CORE_INCS="$CORE_INCS /usr/include/fastcommon /usr/include/fastdfs"
  ```
  
* 如果没有将fast_dfs安装到 /usr/local/bin 下 /usr/local/bin 和 /usr/local/include/ 目录都应该是空的
  
* fastdfs-nginx-module/src/config 定义了 fastdfs-nginx-module 如何安装

## fast-nginx-module 是 nginx 的i一个插件，为nginx提供服务

* 安装nginx所需要的依赖

* yum install -y gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl openssl-devel

* 配置nginx安装信息

  ```shell
./configure --prefix=/usr/local/nginx --pid-path=/var/run/nginx/nginx.pid --lock-path=/var/lock/nginx.lock --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-http_gzip_static_module --http-client-body-temp-path=/var/temp/nginx/client --http-proxy-temp-path=/var/temp/nginx/proxy --http-fastcgi-temp-path=/var/temp/nginx/fastcgi --http-uwsgi-temp-path=/var/temp/nginx/uwsgi --http-scgi-temp-path=/var/temp/nginx/scgi --add-module=/usr/local/fast_dfs/fastdfs-nginx-module/src  
  # --add-module=/usr/local/fast_dfs/fastdfs-nginx-module/src 添加模块

  #最后一行添加模块
  ```
```shell

### 一行代替全部
  
#### 错误多半是之前的nginx 没有完全卸载 rpm -e --nodeps nginx 也没有完全卸载
  
#### rm -rf /usr/lib64/nginx/module  以完全卸载
  
#### cp ngx_http_image_filter_module.so /usr/lib64/nginx/module/ 
  
##  ./configure --prefix=/usr/local/nginx-fastDfs --add-module=/opt/fn/fastdfs-nginx-module/src   （可以成功）

  但是会报错

  nginx: [emerg] module "/usr/lib64/nginx/modules/ngx_http_image_filter_module.so" version 1014001 instead of 1014002 in /usr/share/nginx/modules/mod-http-image-filter.conf:1

  可用教程 https://www.cnblogs.com/zqsb/p/11388889.html

  需要添加 --with-http_image_filter_module=dynamic

  即     ./configure --prefix=/usr/local/nginx-fastDfs --add-module=/opt/fn/fastdfs-nginx-module/src --with-http_image_filter_module=dynamic

  安装依赖库 yum install gd-devel

  编译失败就重新解压，再编译

  ## --add-module 必须定义，此配置信息是用于指定安装nginx时需要加载的模块，如果未指定，nginx安装过程不会加载 fastdfs-nginx-module 模块，后续功能无法实现

  * nginx 可以创建/var/temp/nginx/下的所有子目录，但是不能创建/var/temp/nginx/目录
  
  * mkdir -p /var/temp/nginx/
  
  * make 编译
  
  * make install 安装
  
  * 将fastdfns-nginx-module 的配置文件 fastdfs-nginx-module/src/mod_fastdfs.conf 
  
    拷贝到fast_dfs的配置文件目录下/etc/fdfs/
  
  ```shell
  [root@RabbitMQ_1 src]# cp /opt/nginx/install/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/
```

  

  * 编辑 /etc/fdfs/mod_fastdfs.conf 
  
    * connect_timeout=10 连接超时
    
    * tracker_server=192.168.2.10:22122  #tracker地址
    
    * url_have_group_name = true url中是否需要有卷名(默认false)
    
    * store_path0=/home/fast_dfs_data  服务节点的位置（与storage节点的数据存储节点一致）

### 提供fast_dfs 需要的Http配置文件

* 复制FastDfs/conf/ 安装包中的两个配置文件(http.conf,mime.types)到/etc/fdfs中

  * cp fastdfs/conf/http.conf /etc/fdfs/

  * cp fastdfs/conf/mime.types /etc/fdfs/

### 创建nginx启动所需要的软连接
创建软连接

##### nginx 启动之后，会在默认的 /usr/lib64 目录中查找需要的so文件。如果在安装fastDfs时，修改了make.sh 文件中的TARGET_PREFIX 参数（更改了fastDfs的安装目录），则必须要创建此软连接（默认安装则不需要创建软连接）

ln -s /usr/local/lib64/libfdfsclient.so /usr/lib64/libfdfsclient.so

##### 创建网络存储服务的软连接

在上传文件到FastDfs后，FastDfsClient会返回 group1/M00/00/00/xxxx.xxx 

其中group1时卷名，在mod_fastdfs.conf 配置文件中配置了url_have_group_name = true ，

所以保证URL解析正确。

而其中的M00是fastDfs保存数据时使用的虚拟目录，需要将这个虚拟目录定位到真实数据目录上

ln -s /home/fast_dfs_data/data/ /home/fast_dfs_data/data/M00

<img src="/Users/luo/Library/Application Support/typora-user-images/image-20200622152719303.png" alt="image-20200622152719303" style="zoom: 50%;" />

* 我的理解，要查找group1/M00, 网络地址 group1/ 到了 /home/fast_dfs_data/data 目录之后

* 需要寻找 M00，可以使用图中的软连接直接跳转到本地磁盘，从而继续寻找 00/00/xxx.xx

  

  ### 修改nginx配置文件 

* /etc/fdfs/storage.conf 有 http.server_port = 8888 是 storage 监听的端口

  该存储服务器上Web服务器的端口

  ![image-20200710131842932](/Users/luo/Library/Application Support/typora-user-images/image-20200710131842932.png)

* vi /etc/nginx/nginx.conf server.listen 二者需要保持一致

  ![image-20200710133058979](/Users/luo/Library/Application Support/typora-user-images/image-20200710133058979.png)

  ![image-20200710135629197](/Users/luo/Library/Application Support/typora-user-images/image-20200710135629197.png)

* nginx 还需要有访问Linux 磁盘的权限 root 

  nginx没有权限访问磁盘则404错误

  ![image-20200710135312085](/Users/luo/Library/Application Support/typora-user-images/image-20200710135312085.png)

```shell
# 完整的nginx.conf 配置文件
user  root;		# nginx没有权限访问磁盘则404错误
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8888;
        server_name  192.168.2.10;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location ~/group([0-9])/M00 {
            #root   html;
            #index  index.html index.htm;
            ngx_fastdfs_module;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```



  * 在不重装nginx 的前提下安装module  https://www.cnblogs.com/Leechg/p/9969000.html

  ## 需要重启nginx 、fastdfs 

```shell
# 启动nginx时指定nginx的配置文件
[root@RabbitMQ_1 sbin]# ./nginx -c /etc/nginx/nginx.conf
ngx_http_fastdfs_set pid=52269
```



















| Copy right 2009 Happy Fish / YuQing |                                                              |
| ----------------------------------- | ------------------------------------------------------------ |
|                                     |                                                              |
|                                     | FastDFS may be copied only under the terms of the GNU General |
|                                     | Public License V3, which may be found in the FastDFS source kit. |
|                                     | Please visit the FastDFS Home Page for more detail.          |
|                                     | Chinese language: http://www.fastken.com/                    |
|                                     |                                                              |
|                                     | # step 1. download libfastcommon source codes and install it, |
|                                     | #   github address:  https://github.com/happyfish100/libfastcommon.git |
|                                     | #   gitee address:   https://gitee.com/fastdfs100/libfastcommon.git |
|                                     | # command lines as:                                          |
|                                     |                                                              |
|                                     | git clone https://github.com/happyfish100/libfastcommon.git  |
|                                     | cd libfastcommon; git checkout V1.0.43                       |
|                                     | ./make.sh clean && ./make.sh && ./make.sh install            |
|                                     |                                                              |
|                                     |                                                              |
|                                     | # step 2. download fastdfs source codes and install it,      |
|                                     | #   github address:  https://github.com/happyfish100/fastdfs.git |
|                                     | #   gitee address:   https://gitee.com/fastdfs100/fastdfs.git |
|                                     | # command lines as:                                          |
|                                     |                                                              |
|                                     | git clone https://github.com/happyfish100/fastdfs.git        |
|                                     | cd fastdfs; git checkout V6.06                               |
|                                     | ./make.sh clean && ./make.sh && ./make.sh install            |
|                                     |                                                              |
|                                     |                                                              |
|                                     | # step 3. setup the config files                             |
|                                     | #   the setup script does NOT overwrite existing config files, |
|                                     | #   please feel free to execute this script (take easy :)    |
|                                     |                                                              |
|                                     | ./setup.sh /etc/fdfs                                         |
|                                     |                                                              |
|                                     |                                                              |
|                                     | # step 4. edit or modify the config files of tracker, storage and client |
|                                     | such as:                                                     |
|                                     | vi /etc/fdfs/tracker.conf                                    |
|                                     | vi /etc/fdfs/storage.conf                                    |
|                                     | vi /etc/fdfs/client.conf                                     |
|                                     |                                                              |
|                                     | and so on ...                                                |
|                                     |                                                              |
|                                     |                                                              |
|                                     | # step 5. run the server programs                            |
|                                     | # start the tracker server:                                  |
|                                     | /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart        |
|                                     |                                                              |
|                                     | # start the storage server:                                  |
|                                     | /usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart        |
|                                     |                                                              |
|                                     | # (optional) in Linux, you can start fdfs_trackerd and fdfs_storaged as a service: |
|                                     | /sbin/service fdfs_trackerd restart                          |
|                                     | /sbin/service fdfs_storaged restart                          |
|                                     |                                                              |
|                                     |                                                              |
|                                     | # step 6. (optional) run monitor program                     |
|                                     | # such as:                                                   |
|                                     | /usr/bin/fdfs_monitor /etc/fdfs/client.conf                  |
|                                     |                                                              |
|                                     |                                                              |
|                                     | # step 7. (optional) run the test program                    |
|                                     | # such as:                                                   |
|                                     | /usr/bin/fdfs_test <client_conf_filename> <operation>        |
|                                     | /usr/bin/fdfs_test1 <client_conf_filename> <operation>       |
|                                     |                                                              |
|                                     | # for example, upload a file for test:                       |
|                                     | /usr/bin/fdfs_test /etc/fdfs/client.conf upload /usr/include/stdlib.h |
|                                     |                                                              |
|                                     |                                                              |
|                                     | tracker server config file sample please see conf/tracker.conf |
|                                     |                                                              |
|                                     | storage server config file sample please see conf/storage.conf |
|                                     |                                                              |
|                                     | client config file sample please see conf/client.conf        |
|                                     |                                                              |
|                                     | Item detail                                                  |
|                                     | 1. server common items                                       |
|                                     | ---------------------------------------------------          |
|                                     | \|  item name            \|  type  \| default \| Must \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| base_path             \| string \|         \|  Y   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| disabled              \| boolean\| false   \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| bind_addr             \| string \|         \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| network_timeout       \| int    \| 30(s)   \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| max_connections       \| int    \| 256     \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| log_level             \| string \| info    \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| run_by_group          \| string \|         \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| run_by_user           \| string \|         \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| allow_hosts           \| string \|   *     \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| sync_log_buff_interval\| int    \|  10(s)  \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| thread_stack_size     \| string \|  1M     \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | memo:                                                        |
|                                     | * base_path is the base path of sub dirs:                    |
|                                     | data and logs. base_path must exist and it's sub dirs will   |
|                                     | be automatically created if not exist.                       |
|                                     | $base_path/data: store data files                            |
|                                     | $base_path/logs: store log files                             |
|                                     | * log_level is the standard log level as syslog, case insensitive |
|                                     | # emerg: for emergency                                       |
|                                     | # alert                                                      |
|                                     | # crit: for critical                                         |
|                                     | # error                                                      |
|                                     | # warn: for warning                                          |
|                                     | # notice                                                     |
|                                     | # info                                                       |
|                                     | # debug                                                      |
|                                     | * allow_hosts can ocur more than once, host can be hostname or ip address, |
|                                     | "*" means match all ip addresses, can use range like this: 10.0.1.[1-15,20] |
|                                     | or host[01-08,20-25].domain.com, for example:                |
|                                     | allow_hosts=10.0.1.[1-15,20]                                 |
|                                     | allow_hosts=host[01-08,20-25].domain.com                     |
|                                     |                                                              |
|                                     | 2. tracker server items                                      |
|                                     | ---------------------------------------------------          |
|                                     | \|  item name            \|  type  \| default \| Must \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| port                  \| int    \| 22000   \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| store_lookup          \| int    \|  0      \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| store_group           \| string \|         \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| store_server          \| int    \|  0      \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| store_path            \| int    \|  0      \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| download_server       \| int    \|  0      \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     | \| reserved_storage_space\| string \|  1GB    \|  N   \|     |
|                                     | ---------------------------------------------------          |
|                                     |                                                              |
|                                     | memo:                                                        |
|                                     | * the value of store_lookup is:                              |
|                                     | 0: round robin (default)                                     |
|                                     | 1: specify group                                             |
|                                     | 2: load balance (supported since V1.1)                       |
|                                     | * store_group is the name of group to store files.           |
|                                     | when store_lookup set to 1(specify group),                   |
|                                     | store_group must be set to a specified group name.           |
|                                     | * reserved_storage_space is the reserved storage space for system |
|                                     | or other applications. if the free(available) space of any stoarge |
|                                     | server in a group <= reserved_storage_space, no file can be uploaded |
|                                     | to this group (since V1.1)                                   |
|                                     | bytes unit can be one of follows:                            |
|                                     | # G or g for gigabyte(GB)                                    |
|                                     | # M or m for megabyte(MB)                                    |
|                                     | # K or k for kilobyte(KB)                                    |
|                                     | # no unit for byte(B)                                        |
|                                     |                                                              |
|                                     | 3. storage server items                                      |
|                                     | -------------------------------------------------            |
|                                     | \|  item name          \|  type  \| default \| Must \|       |
|                                     | -------------------------------------------------            |
|                                     | \| group_name          \| string \|         \|  Y   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| tracker_server      \| string \|         \|  Y   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| port                \| int    \| 23000   \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| heart_beat_interval \| int    \|  30(s)  \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| stat_report_interval\| int    \| 300(s)  \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| sync_wait_msec      \| int    \| 100(ms) \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| sync_interval       \| int    \|   0(ms) \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| sync_start_time     \| string \|  00:00  \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| sync_end_time       \| string \|  23:59  \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| store_path_count    \| int    \|   1     \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| store_path0         \| string \|base_path\|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| store_path#         \| string \|         \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \|subdir_count_per_path\| int    \|   256   \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \|check_file_duplicate \| boolean\|    0    \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| key_namespace       \| string \|         \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| keep_alive          \| boolean\|    0    \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     | \| sync_binlog_buff_interval\| int \|   60s \|  N   \|       |
|                                     | -------------------------------------------------            |
|                                     |                                                              |
|                                     | memo:                                                        |
|                                     | * tracker_server can ocur more than once, and tracker_server format is |
|                                     | "host:port", host can be hostname or ip address.             |
|                                     | * store_path#, # for digital, based 0                        |
|                                     | * check_file_duplicate: when set to true, must work with FastDHT server, |
|                                     | more detail please see INSTALL of FastDHT. FastDHT download page: |
|                                     | http://code.google.com/p/fastdht/downloads/list              |
|                                     | * key_namespace: FastDHT key namespace, can't be empty when  |
|                                     | check_file_duplicate is true. the key namespace should short as possible |