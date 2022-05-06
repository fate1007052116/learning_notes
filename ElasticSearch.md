# ElasticSearch

## 一、运行

> 注意：最新的elasticSearch 7.9.2运行时环境要求是jdk15
>
> 官方下载`elasticSearch`默认附赠jdk15
>
> 华为镜像下载
>
> ElasticSearch: https://mirrors.huaweicloud.com/elasticsearch/?C=N&O=D
> logstash: https://mirrors.huaweicloud.com/logstash/?C=N&O=D
> kibana: https://mirrors.huaweicloud.com/kibana/?C=N&O=D

### 1.创建elsearch用户组及elsearch用户：

```shell
groupadd elsearch
useradd elsearch -g elsearch
passwd elsearch
```

### 2. 更改elasticsearch文件夹及内部文件的所属用户及组为elsearch:elsearch

```shell
[root@localhost elasticsearch]# ls
elasticsearch-7.9.2
[root@localhost elasticsearch]# chown -R elsearch:elsearch elasticsearch-7.9.2/
```

### 3. 切换到elsearch用户再启动

```shell
[root@localhost elasticsearch]# su elsearch
[elsearch@localhost elasticsearch]$ cd /opt/elasticsearch/elasticsearch-7.9.2/bin/
[elsearch@localhost bin]$ ./elasticsearch
```

### 4. 如果是mac系统，需要放在mac原生文件系统中

```shell
# 如果放在exfat 中则抛以下异常
java.nio.channels.OverlappingFileLockException
```

### 5.访问端口

#### （1）http端口

![Snip20201002_1](/Users/luo/Documents/开发笔记/images/Snip20201002_1.png)

#### （2）http访问非http端口

![Snip20201002_2](/Users/luo/Documents/开发笔记/images/Snip20201002_2.png)

#### （3）查看es集群节点

>Localhost:9200/_cat/nodes

### 6. nodeJs 使用淘宝npm源

> https://developer.aliyun.com/mirror/NPM?from=tnpm

```shell
# 全局安装 cnpm
npm install -g cnpm --registry=https://registry.npm.taobao.org

# 为cnpm添加别名
alias cnpm="npm --registry=https://registry.npm.taobao.org \
--cache=$HOME/.npm/.cache/cnpm \
--disturl=https://npm.taobao.org/dist \
--userconfig=$HOME/.cnpmrc"

# 使用淘宝的npm源安装依赖
cnpm install
```



### 7.安装可视化界面

> https://github.com/mobz/elasticsearch-head

```shell
npm install
npm start
```

### 8. Elastic-search 开启跨域

在`config/elasticsearch.yml`的最后一行添加允许跨域

```shell
luo@luodeMacBook-Pro config % pwd
/Volumes/OS/environment/elasticsearch-7.9.2/config
luo@luodeMacBook-Pro config % tail elasticsearch.yml
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true

http:
  cors:
    enabled: true
    allow-origin: "*"
  host: 0.0.0.0 # 允许任何主机访问 ES  ，不然就只有localhost可以访问
```



### 9.ELK

> Log stash（收集清洗数据）
>
> Elastic search （搜索，存储）
>
> kibana（展示数据）

![Snip20201002_3](/Users/luo/Documents/开发笔记/images/Snip20201002_3.png)

### 10.安装和汉化Kibana

> 注意：Kibana 版本要和 elastic search 版本一致

kibana 下载好之后就可以执行`bin/kibana`来启动整个`kibana`

kibana监听的端口 `http://localhost:5601/`

`/kibana-7.8.0-darwin-x86_64/x-pack/.i18nrc.json`是用于国际化的文件

`kibana-7.8.0-darwin-x86_64/x-pack/plugins/translations/translations/zh-CN.json`是汉化文件

汉化开始

`kibana-7.8.0-darwin-x86_64/config/kibana.yml`末尾添加`i18n.locale: "zh-CN"%`即可使用中文

```shell
luo@luodeMacBook-Pro config % tail -5 kibana.yml

# Specifies locale to be used for all localizable strings, dates and number formats.
# Supported languages are the following: English - en , by default , Chinese - zh-CN .
#i18n.locale: "en"
i18n.locale: "zh-CN"%
```

>注意

在`kibana-7.8.0-darwin-x86_64/config/kibana.yml`中修改`kibana`连接`elasticsearch`的地址，修改不了

```shell
#elasticsearch.hosts: ["http://localhost:9200"]
```

`kibana`容器中查看环境变量，重启后失效，删掉容器，重新设置esip地址

```shell
bash-4.2$ export
declare -x ELASTICSEARCH_HOSTS="http://172.17.0.1:9200"
declare -x ELASTIC_CONTAINER="true"
declare -x HOME="/usr/share/kibana"
declare -x HOSTNAME="f8bcd570684c"
declare -x OLDPWD
declare -x PATH="/usr/share/kibana/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
declare -x PWD="/usr/share/kibana"
declare -x SHLVL="1"
declare -x TERM="xterm"

# 重新设置elasticsearch的环境变量
bash-4.2$ unset ELASTICSEARCH_HOSTS
bash-4.2$ export ELASTICSEARCH_HOSTS="http://192.168.200.1:9200"

bash-4.2$ export
declare -x ELASTICSEARCH_HOSTS="http://192.168.200.1:9200"
declare -x ELASTIC_CONTAINER="true"
declare -x HOME="/usr/share/kibana"
declare -x HOSTNAME="f8bcd570684c"
declare -x OLDPWD
declare -x PATH="/usr/share/kibana/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
declare -x PWD="/usr/share/kibana"
declare -x SHLVL="1"
declare -x TERM="xterm"
```



## 二、elastic-search 核心概念

> elastic search 是面向文档，关系型数据库和 elastic search 客观的对比
>
> elastic search 中一切数据都是json
>
> 1. 索引
>
> 2. 字段类型（mapping映射）
> 3. 文档
> 4. 底层使用`lucene`倒排索引分片

| realtional DB    | Elastic search            |
| ---------------- | ------------------------- |
| 数据库(database) | 索引(index)就和数据库一样 |
| 表(tables)       | types（主键被弃用）       |
| 行(rows)         | Document                  |
| 字段(columns)    | Fields                    |

Elastic search （集群）中可以包含多个索引（数据库），每个索引中可以包含多个类型（表），每个类型下又可以包含多个文档（行），每个文档中又包含多个属性（列）

### 1.物理设计

elastic-search 在后台把每个索引划分成多个分片，每分分片可以在集群中的不同服务器之间迁移

> Elastic-search 一个节点也是一个集群

![Snip20201002_4](/Users/luo/Documents/开发笔记/images/Snip20201002_4.png)

![Snip20201002_5](/Users/luo/Documents/开发笔记/images/Snip20201002_5.png)

### 2.逻辑设计

一个索引类型中，包含多个文档，比如说文档一，文档二，当我们要索引一篇文档的时候，可以通过这样的一个顺序找打它

> 索引 > 类型 > 文档id，通过这个组合我们就能索引到某个具体的文档
>
> 注意：id不必是整数，实际上它是个字符串

> 文档

之前说elastic-search 是面向文档的，那么就意味着索引和搜索数据的最小单位就是文档，elastic-search中，文档有几个重要的属性

* 自我包含：一篇文档同时包含字段和其对应的值，也就是同时包含`key:value`

* 可以是层次型的：一个文档中包含子文档，复杂的逻辑实体就是这么来的，就是一个json对象，fastjson进行自动转换

* 灵活的结构：文档不依赖于预先定义的模式，我们知道关系型数据库中，要提前定义字段才能使用，在elastic-search中，对于字段是非常灵活的，有些时候我们可以忽略该字段，或者动态的添加一个新的字段

  尽管我们可以随意的新增或者忽略某个字段，但是每个字段的类型非常重要，比如一个年龄字段的类型，可以是字符串也可以是整形。因为elastic-search 会保存字段和类型之间的映射及其他的设置。这种映射具体到每个映射的每种类型，这也是为什么在elastic-search中，类型有些时候也被称为映射类型

> 类型

类型就是文档的逻辑容器，就像关系型数据库一样，表格是行的容器。类型中对于字段的定义称为映射，比如`name`映射为字符串类型。我们说文档是无模式的，他们不需要拥有映射中所定义的所有字段，比如新增加一个字段，那么`elastic search`是怎么做的呢？`elastic search`会自动的将新字段加入映射，但是这个字段不确定他是什么类型，`elastic search`就开始猜，比如这个值是`18`，那么`elastic search`就会认为他是整型。但是`elastic search`也可能猜不对，所以最安全的方式就是提前定义好所需要的映射，这点根关系型数据库就殊途同归了，先定义好字段，然后再使用，别整什么幺蛾子

> 索引

就是数据库！索引是映射类型的容器，`elastic search`中的索引是一个非常大的文档集合。索引存储了映射类型的字段和其他设置。然后他们被存储到各个分片上了。

### 3.物理设计

#### （1）节点和分片如何工作

一个集群至少有一个节点，而一个节点就是一个`elastic search`进程，节点可以有多个索引默认的，如果你创建索引，那么索引将会由5个分片（primary shard，又称主分片）构成，每一个主分片都会有一个副本（replica shard，又称复制分片）

![Snip20201002_6](/Users/luo/Documents/开发笔记/images/Snip20201002_6.png)

上图是一个有三个节点的集群，可以看到主分片和对应的复制分片都不会在同一个节点内，这样有利于某个节点挂掉了，数据也不至于丢失。实际上，一个分片是一个`Lucene`索引，一个包含倒排索引的文件目录，倒排索引的结构使得`elastic search`在不扫描全部文档的情况下，就能告诉你哪些文档包含特定的关键字。

### 4.倒排索引

`elastic search`使用的是一种称为倒排索引的结构，采用`Lucene`倒排索引作为底层。这种结构适用于快速的全文搜索，一个索引由文档中所有不重复的列表构成，对于每一个词，都有一个包含它的文档列表。例如，现在有两个文档，每个文档有以下内容：

```
study every day, good good up to forever # 文档一包含的内容
to forever, study every day, good good up # 文档二包含的内容
```

为了创建倒排索引，我们首先要将每个文档拆分成独立的词（或称为词条或者tokens），然后创建一个包含所有不重复的词条和排序列表，然后列出每个词条出现在哪一个文档

| Term    | doc1 | doc2 |
| ------- | ---- | ---- |
| Study  | y |      |
| To     |  |      |
| every   | y | y |
| forever | y | y |
| day     | y | y |
| study   |      | y |
| good    | y | y |
| every   | y | y |
|to|y||
|up|y|y|
现在我们试图搜索`to forever`，只需要查看包含每个词条的文档

| term | doc1 | doc2 |
| ---- | ---- | ---- |
| to   |  y    |      |
|forever|y|y|
|total（匹配率，权重）|2|1|

两个文档都匹配，但是第一个文档比第二个匹配度更高。如果没有特别的条件，现在这两个包含关键字的文档都将被返回。



再来看一个实例，比如我们通过博客标签来搜索博客文章，那么倒排索引列表就是这样一个结构

| 博客文章（原始数据） |               | 索引列表（倒排索引） |            |
| -------------------- | ------------- | -------------------- | ---------- |
| 博客文章id           | 标签          | 标签                 | 博客文章id |
| 1                    | python        | pythod               | 1,2,3      |
| 2                    | python        | linux                | 3,4        |
| 3                    | Linux, python |                      |            |
| 4                    | linux         |                      |            |

如果要搜索含有`python`标签的文章，那么相对于查找所有原始数据而言，查找倒排索引后的数据将会快的多。只需要查看标签这一栏，然后获取文章id即可。完全过滤掉所有无关的数据，提高效率

`elastic search`的索引和`Lucene`的索引对比

在`elastic search`中，索引这个词被频繁使用，这就是术语的使用。在`elastic search`中，索引被分为多个分片，每份分片是一个`lucene`的索引。==所以一个`elastic search`索引是由多个`lucene`索引组成的。==别问为什么，谁让`elastic search`使用`lucene`作为底层呢！如无特指，说索引都是指`elastic search`的索引

## 三、IK分词器

> 什么是IK分词器？

分词：即把一段中文或者别的字符串划分为一个个关键字，我们在搜索的时候会把自己的信息进行分词，会把数据库中或者索引库中的数据进行分词，然后进行一个匹配操作，默认的中文分词是将每个字看成一个词，比如`我爱狂神`会被分为`我` `爱` `狂` `神`，这显然是不符合要求的，所以我们需要安装中文分词器ik来解决这个问题

如果要使用中文搜索，建议使用ik分词器

IK提供了两个分词算法：`ik_smart`和`ik_max_word`，其中`ik_smart`为最少切分，`ik_max_word`为最细粒度划分

### 1.安装ik分词器

> https://github.com/medcl/elasticsearch-analysis-ik

下载完毕之后放到`elastic search/plugins`插件文件夹中即可

![Snip20201002_9](/Users/luo/Documents/开发笔记/images/Snip20201002_9.png)

重启`elastic search`，要先打开`elastic search`再开启`kibana`

```shell
# 可以看到，插件被加载了
[2020-10-02T16:16:59,317][INFO ][o.e.p.PluginsService     ] [luodeMacBook-Pro.local] loaded plugin [analysis-ik]
```

也可以使用`elastic search `自带的命令`elasticsearch-plugin`来查看安装了哪些插件

```shell
luo@luodeMacBook-Pro bin % pwd
/Volumes/OS/environment/elasticsearch-7.9.2/bin
# mac下，前提要删除 /elasticsearch-7.9.2/plugins/.DS_Store 这个文件，
# 不然elastic search 会试图去解析 .DS_Store 这个插件
luo@luodeMacBook-Pro bin % ./elasticsearch-plugin list
ik
```

### 2.查看不同ik分词器的效果

`ik_smart`最少切分分词器

![Snip20201002_11](/Users/luo/Documents/开发笔记/images/Snip20201002_11.png)

`ik_max_word`最细粒度划分，穷尽词库的可能

又一个字典

![Snip20201002_10](/Users/luo/Documents/开发笔记/images/Snip20201002_10.png)

### 3. 在ik分词器中增加自己的字典

![Snip20201002_12](/Users/luo/Documents/开发笔记/images/Snip20201002_12.png)

发现问题：狂神说被分词器拆开了；

这种自己需要的词，需要我们自己加到分词器字典中

> ik分词器中添加自己的关键字
>
> `elasticsearch-7.9.2/plugins/ik/config/IKAnalyzer.cfg.xml`

```shell
# 在ik分词器插件中进行配置
luo@luodeMacBook-Pro config % pwd
/Volumes/OS/environment/elasticsearch-7.9.2/plugins/ik/config
# 创建自定义字典，编码必须是utf-8才能起效
luo@luodeMacBook-Pro config % vi kuangshen.dic
# 往字典中添加自定义的关键字
luo@luodeMacBook-Pro config % cat kuangshen.dic
狂神说

```

![Snip20201002_14](/Users/luo/Documents/开发笔记/images/Snip20201002_14.png)

然后重启`elastic search`

```shell
# 已经成功加载字典
[2020-10-02T17:40:39,575][INFO ][o.w.a.d.Dictionary       ] [luodeMacBook-Pro.local] [Dict Loading] /Volumes/OS/environment/elasticsearch-7.9.2/plugins/ik/config/kuangshen.dic
```



![Snip20201002_15](/Users/luo/Documents/开发笔记/images/Snip20201002_15.png)

### 4.使用nginx代理远程分词器

看`docker.md`的`安装nginx并反向代理elastic search 分词器`

## 四、关于索引的基本操作

一种软件架构风格，而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存机制

### 1.基本rest命令说明

| method | url地址                                         | 描述                       |
| ------ | ----------------------------------------------- | -------------------------- |
| put    | Localhost:9200/索引名称/类型名称/文档id         | 创建文档（指定文档id）     |
| post   | Localhost:9200/索引名称/类型名称                | 创建文档（随机指定文档id） |
| post   | Localhost:9200/索引名称/类型名称/文档id/_update | 修改文档                   |
| delete | Localhost:9200/索引名称/类型名称/文档id         | 删除文档                   |
| get    | Localhost:9200/索引名称/类型名称/文档id         | 通过文档id查询文档         |
| post   | Localhost:9200/索引名称/类型名称/_search        | 查询所有数据               |

（1）查看节点的健康状况

![Snip20201003_4](/Users/luo/Documents/开发笔记/images/Snip20201003_4.png)

（2）查看es健康状况

![Snip20201003_5](/Users/luo/Documents/开发笔记/images/Snip20201003_5.png)

（3）查看主节点

![Snip20201003_6](/Users/luo/Documents/开发笔记/images/Snip20201003_6.png)

（4）查看所有索引

![Snip20201003_7](/Users/luo/Documents/开发笔记/images/Snip20201003_7.png)

### 2.同时创建索引和文档

```javascript
put /索引名/类型名（未来类型字段将被删除）/文档id
{
	请求体:""
}
```



![Snip20201002_16](/Users/luo/Documents/开发笔记/images/Snip20201002_16.png)



![Snip20201002_17](/Users/luo/Documents/开发笔记/images/Snip20201002_17.png)

> 那么name这个属性用不用指定类型呢

* 字符串类型：text	keyword（keyword为关键字，不可分割）
* 数值类型：long    integer     short     byte.   Double   float.  Half   scaled   
* 日期类型：date
* 布尔值类型：boolean
* 二进制类型：binary
* 等等....

### 3.创建索引的时候指定属性映射

```json
PUT /index-2
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text"
      },
      "age":{
        "type": "integer"
      },
      "birthday":{
        "type": "date"
      }
    }
  }
}
```

![Snip20201002_18](/Users/luo/Documents/开发笔记/images/Snip20201002_18.png)

### 4.得到当前索引的信息

![Snip20201002_19](/Users/luo/Documents/开发笔记/images/Snip20201002_19.png)

### 5.插入默认文档类型

> 使用`_doc`替换`/索引名/类型名（未来类型字段将被删除）/文档id`中的`类型名`

![Snip20201002_21](/Users/luo/Documents/开发笔记/images/Snip20201002_21.png)

发现属性映射已经自动做好

> 如果插入文档的时候没有指定属性映射，那么es就会默认为我们配置字段类型

![Snip20201002_22](/Users/luo/Documents/开发笔记/images/Snip20201002_22.png)

### 6.扩展命令

> 扩展：通过es命令获取索引的情况！
>
> 通过 GET _cat 可以获得es的当前很多信息

显示es健康状态

```shell
GET _cat/health
```

![Snip20201002_23](/Users/luo/Documents/开发笔记/images/Snip20201002_23.png)

查询所有索引的信息

```shell
GET _cat/indices
```

![Snip20201002_24](/Users/luo/Documents/开发笔记/images/Snip20201002_24.png)

### 7.修改api

#### （1）通过覆盖来实现修改

> 修改：提交的还是使用put即可！id相同，然后覆盖

原有数据

![Snip20201002_25](/Users/luo/Documents/开发笔记/images/Snip20201002_25.png)

执行覆盖操作（同一索引，同一文档id）

> 注意：这样覆盖了之后，右边的`_version`版本号会增加
>
> `result`也会显示为`updated`

![Snip20201002_26](/Users/luo/Documents/开发笔记/images/Snip20201002_26.png)

文档覆盖之后

![Snip20201002_27](/Users/luo/Documents/开发笔记/images/Snip20201002_27.png)

相当于做了一次更新

#### （2）修改（将被弃用）

> 修改：`_version`会更新
>
> `result`的状态为`result`

![Snip20201002_28](/Users/luo/Documents/开发笔记/images/Snip20201002_28.png)

#### （3）修改

```json
POST /my-index-4/_update/1/
{
  "doc":{
      "name":"大家"
  }
}
```

![Snip20201002_29](/Users/luo/Documents/开发笔记/images/Snip20201002_29.png)

### 8.删除

#### （1）删除索引

> DELETE /索引名称
>
> DELETE /索引名称/_doc/文档id
>
> 通过delete命令实现删除，根据你的请求来判断删除的是索引还是记录

![Snip20201002_30](/Users/luo/Documents/开发笔记/images/Snip20201002_30.png)

#### （2）删除索引中的文档（弃用）

![Snip20201002_31](/Users/luo/Documents/开发笔记/images/Snip20201002_31.png)

#### （3）删除索引中的文档

![Snip20201002_32](/Users/luo/Documents/开发笔记/images/Snip20201002_32.png)

### 9.为已存在的索引添加属性，并制定属性的映射

只能用于添加字段

```json
  PUT /luo-simple/_mapping
  {
    "properties":{
      "employee-id":{
        "type":"keyword",
        "index":"false"
      }
    }
  }
```

### 10.更新映射

对于已经存在的映射字段，我们不能更新，更新必须创建新的索引进行数据迁移

```json
// 创建一个新的索引，并且指定映射
PUT new-bank/
{
  "mappings": {
    "properties": {
      "account_number": {
        "type": "long"
      },
      "address": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "age": {
        "type": "long"
      },
      "balance": {
        "type": "long"
      },
      "city": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "email": {
        "type": "keyword"
      },
      "employer": {
        "type": "keyword"
      },
      "firstname": {
        "type": "keyword"
      },
      "gender": {
        "type": "keyword"
      },
      "lastname": {
        "type": "keyword"
      },
      "state": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```

### 11.数据迁移

先要创建出索引的正确映射

```json
POST _reindex
{
  "source": {
    "index": "luo"
  },
  "dest": {
    "index": "new-bank"
  }
}
```





## 五、关于文档的操作

>文档的基本操作

### 1.插入文档

#### （1）put插入（弃用）

> 这个插入，如果id已经存在，则会==完全覆盖==现有文档
>
> 即，若旧文档中的属性值，在覆盖的时候没有指定，那么覆盖之后，老文档中的值就没有了，比较恐怖
>
> 不传值，该属性就会被设置为null

![Snip20201002_34](/Users/luo/Documents/开发笔记/images/Snip20201002_34.png)

#### （2）put插入

> 注意：这个插入不能覆盖现有的文档（如果文档id已经存在则插入失败）
>
> 所有`put`方式都必须携带id

![Snip20201002_35](/Users/luo/Documents/开发笔记/images/Snip20201002_35.png)

文档已经存在，插入失败

![Snip20201002_36](/Users/luo/Documents/开发笔记/images/Snip20201002_36.png)

#### （3）post方式不带id插入

同一请求发送多次，将产生不同id，不能覆盖

![Snip20201003_8](/Users/luo/Documents/开发笔记/images/Snip20201003_8.png)

#### （4）post带id插入

可以覆盖

![Snip20201003_9](/Users/luo/Documents/开发笔记/images/Snip20201003_9.png)

### 2.获取文档

> GET /索引名称/_doc/文档id
>
> `_seq_no` 并发控制字段，每次更新就会+1，用来做乐观锁
>
> `_primary_term` 同上，主分片重新分配，如果重启，就会变化

![Snip20201002_33](/Users/luo/Documents/开发笔记/images/Snip20201002_33.png)

### 3.更新文档

#### （1）post+_update更新方式

>只会更新涉及到的属性（局部更新），此次更新没有涉及到的属性将会保持不变
>
>一旦使用了`_update`作为更新url，就必须在请求体中将要更新的数据放入`doc:{}`对象中
>
>```json
>POST /luo/_update/3
>{
>"doc":{
>  "name":"大肥燕"
>  }
>}
>```

![Snip20201002_37](/Users/luo/Documents/开发笔记/images/Snip20201002_37.png)

> 注意：如果此次更新没有数据被修改，结果就是`noop(no operation)`，而且`version`，`_seq_no`，`_primary_term`的值都不会变更
>
> `post`+`_update`方式会对比原来的数据，如果此次更新的数据和原来的一样就什么都不做
>
> 如果要更新的文档不存在，就会报错

![Snip20201004_6](/Users/luo/Documents/开发笔记/images/Snip20201004_6.png)

#### （2）post+_doc方式

> `_doc`方式的更新会覆盖原来的所有属性（全局更新），此次更新没有涉及到的属性将会被清空
>
> 不会去检查此次更新的数据和原数据是否一致，`version`和`_seq_no`会一直往上+1
>
> `_primary_term`的值无论数据是否更新都不会变更
>
> 注意：`_doc`方式更新在请求体中==不==能写`"doc":{}`
>
> 

![Snip20201004_7](/Users/luo/Documents/开发笔记/images/Snip20201004_7.png)

#### （3）乐观锁修改

注意更新之前的`_seq_no`和`_primary_term`

![Snip20201004_2](/Users/luo/Documents/开发笔记/images/Snip20201004_2.png)

A客户端模拟并发更新操作

![Snip20201004_3](/Users/luo/Documents/开发笔记/images/Snip20201004_3.png)

B客户端模拟并发更新操作

![Snip20201004_4](/Users/luo/Documents/开发笔记/images/Snip20201004_4.png)

这样就只有A更新成功

B如果还想要更新，就必须获取到最新的两个参数`_seq_no`和`_primary_term`

![Snip20201004_5](/Users/luo/Documents/开发笔记/images/Snip20201004_5.png)

这样B也更新了

### 4.搜索

简单的查询，可以根据默认的映射规则，产生基本的查询

查询出来的结果带有`_score`，这是匹配度，匹配度越高则分值越高

#### （1）推荐搜索

> GET /索引名字/_search?q=key:value
>
> q在这里代指query

![Snip20201002_38](/Users/luo/Documents/开发笔记/images/Snip20201002_38.png)

#### （2）搜索（被弃用）

![Snip20201002_39](/Users/luo/Documents/开发笔记/images/Snip20201002_39.png)

### 5.复杂搜索

#### （0）查询

1. 简单查询

上面简单查询的完整版

> Hit包含
>
> 索引和文档的信息
>
> 查询结果的总数
>
> 查询出来具体的文档
>
> 数据中的东西都可以遍历出来了
>
> 分数：我们可以通过分数来判断谁更加精确，更符合结果

当`match`下要匹配的属性是数字的时候就是精确匹配，如果是字符串的时候就是模糊匹配

全文检索按照评分`_score`进行排序，会对检索条件进行分词匹配

![Snip20201002_40](/Users/luo/Documents/开发笔记/images/Snip20201002_40.png)

2. 短句查询

查询的内容`mill lane`不会被分词器分词，而是直接作为一个整体去查询

```json
GET /luo/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

3. 多属性匹配

```json
// 只要 address 或者 city 属性中含有 Mill 关键字，就可以被查出
GET /luo/_search
{
  "query": { 
    "multi_match": {
      "query": "Mill",
      "fields": ["address","city"] 
    }
  }
}
```

4. 多属性多值匹配

```json
// 属性"address","city" 中任意一个含有 Mill 或者 Lane （总共四种组合），就能匹配上
GET /luo/_search
{
  "query": { 
    "multi_match": {
      "query": "Mill Lane",
      "fields": ["address","city"]
    }
  }
}
```

5. bool复杂查询

#### （1）结果过滤

> 使用_source显示指定的属性
>
> 作用类似于mysql中的select

```json
GET /luo/_search
{
  "query": {
    "match": {
      "name": "罗"
    }
  },
  "_source": ["name","age"]
}
```



![Snip20201002_41](/Users/luo/Documents/开发笔记/images/Snip20201002_41.png)

#### （2）结果排序

>一旦使用了排序之后`_score`就为null

```json
GET /luo/_search
{
  "query": {
    "match": {
      "name": "罗"
    }
  },
  "sort": [
    {
      "age": {					// 要排序的属性
        "order": "desc" //asc 是生序，desc是降序
      }
    }
  ]
}
```

![Snip20201002_42](/Users/luo/Documents/开发笔记/images/Snip20201002_42.png)

双重结果排序

一级结果排序相同时，在根据二级结果排序规则来排序

```json
GET /luo/_search
{
  "query": {
    "match": {
      "name": "罗"
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    },{
      "name":{					// 这里只是为了演示二级排序，test类型的属性不能用来排序
        "order": "desc"	
      }
    }
  ]
}
```

#### （3）分页

> From 从那一条记录开始，数据索引下标从0开始
>
> size 每页显示几条文档
>
> 类似mysql的 limit startIndex size

```json
GET /luo/_search
{
  "query": {
    "match": {
      "name": "罗"
    }
  },
  "sort": [
    {
      "age": {
        "order": "desc"
      }
    },{
      "name":{
        "order": "desc"
      }
    }
  ]
}
```

![Snip20201002_44](/Users/luo/Documents/开发笔记/images/Snip20201002_44.png)

#### （4）多条件匹配

##### 		1.must

>条件都要满足才能查询出来
>
>Must 相当于 mysql 中的 and 条件

```json
GET /luo/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name":"aaa"
          }
          
        },{
          "match": {
            "age": 22
          }
        }
      ]
    }
  }
 
}
```

对于英文来说是精确匹配

![Snip20201002_46](/Users/luo/Documents/开发笔记/images/Snip20201002_46.png)

查询条件中少了一个a就查不到了

![Snip20201002_47](/Users/luo/Documents/开发笔记/images/Snip20201002_47.png)

对中文来说，却又是模糊匹配

![Snip20201002_47](/Users/luo/Documents/开发笔记/images/Snip20201002_47.png)

##### 	2.should

>条件都要满足一个就能查询出来
>
>should 相当于 mysql 中的 or 条件
>
>不满足`should`也没有关系，只不过满足了`should`，`_score`的得分会更高

```json
GET /luo/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "name":"罗"
          }
          
        },{
          "match": {
            "age": 18
          }
        }
      ]
    }
  }
}
```

![Snip20201002_49](/Users/luo/Documents/开发笔记/images/Snip20201002_49.png)

##### 3.must_not

> 查询出来的结果都不能满足
>
> 名字不能含有`罗`而且年龄不能是`3`岁

```json
GET /luo/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "match": {
            "name":"罗"
          }
          
        },{
          "match": {
            "age": 18
          }
        }
      ]
    }
  }
}
```

![Snip20201002_51](/Users/luo/Documents/开发笔记/images/Snip20201002_51.png)

##### 4.混合

>`must`和`should`条件满足`_score`的得分都会增加
>
>`must_not`会被当成是一个`filter`，不会增加`_score`

```json
GET /luo/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "gender": "M"
          }
        },
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "must_not": [
        {"match": {
          "age": "18"
        }}
      ],
      "should": [
        {
          "match": {
            "lastname": "Wallace"
          }
        }
      ]
    }
  }
}
```



#### （5）过滤

>`filter`中的条件可以放在`query`中
>
>`filter`和`query`的区别是，在`filter`中的条件不会纳入`_score`的计算范围

```json
GET luo/_search
{
  "query": {
    "bool": {
      "filter": [
        {"range": {
          "age": {
            "gte": 18,
            "lte": 20
          }
        }}
      ]
    }
  }
}
```

`query`中的`filter`不会被计算得分（即使在`bool`中组合其他条件`filter`也不会有得分）

![Snip20201007_18](/Users/luo/Documents/开发笔记/images/Snip20201007_18.png)

`query`中`filter`能做的，其他条件也能做，而且还能额外计算`_score`

![Snip20201007_19](/Users/luo/Documents/开发笔记/images/Snip20201007_19.png)



#### （6）多条件模糊查询

> 多个条件之间用空格隔开
>
> 只要满足其中一个结果就可以被查出
>
> 这时候就可以通过`_score`来进行判断

![Snip20201002_54](/Users/luo/Documents/开发笔记/images/Snip20201002_54.png)

#### （7）精确查询

`term`查询是直接通过倒排索引指定词条进行精确的查找

> 关于分词：
>
> term，直接查找精确的值（贼快，效率高）
>
> match，会使用分词器进行解析（中文不是精确查找，英文是精确查找），先通过分词器分析文档，然后再通过文档进行精确查询
>
> 两个数据类型：text keyword
>
> text会被分词器解析，keyword不会被分词器解析

##### 1.创建测试数据库

![Snip20201002_55](/Users/luo/Documents/开发笔记/images/Snip20201002_55.png)

`keyword`类型的字符串不会被ik分词器解析

![Snip20201002_56](/Users/luo/Documents/开发笔记/images/Snip20201002_56.png)

不是keyword类型的字符串就被分词器解析

![Snip20201002_57](/Users/luo/Documents/开发笔记/images/Snip20201002_57.png)

`test-db`索引中的数据

![Snip20201002_58](/Users/luo/Documents/开发笔记/images/Snip20201002_58.png)

##### 2.name是`text`类型，所以会被分词器解析

![Snip20201002_59](/Users/luo/Documents/开发笔记/images/Snip20201002_59.png)

##### 3.desc是keyword类型，不会被分词器解析，所以做精确匹配

![Snip20201002_60](/Users/luo/Documents/开发笔记/images/Snip20201002_60.png)

##### 4.精确查询查询多个条件

![Snip20201002_62](/Users/luo/Documents/开发笔记/images/Snip20201002_62.png)

##### 5.term

**和match一样匹配某个属性的值，全文检索用match，其他非text字段用term**

term精确查找

> 只有`elasticSearch`中保存的该属性是`keyword`类型，才能使用`term`来精确匹配不可分割的字符串

```json
// 创建索引的时候指定属性name的映射是keyword
PUT luo-keyword
{
  "mappings": {
    "properties": {
      "name":{
        "type": "keyword"
      }
    }
  }
}
// 插入记录
PUT luo-keyword/_doc/1
{
  "name":"罗俊华"
}
// 发现keyword + term 可以做精确匹配
GET luo-keyword/_search
{
  "query": {
    "term": {
      "name": {
        "value": "罗俊华"
      }
    }
  }
}
```

![Snip20201007_20](/Users/luo/Documents/开发笔记/images/Snip20201007_20.png)

`match_phrase`查询短句的方式达到精确查询的效果

```json
GET luo/_search
{
  "query": {
    "match_phrase": {
      "address": "132 Gunnison Court"
    }
  }
}

// 因为是短句匹配，只要短句匹配上了就是查到了
GET luo/_search
{
  "query": {
    "match_phrase": {
      "address": "132 Gunnison" // 所以删掉一个单词依然可以匹配上
    }
  }
}
```

使用`.keyword`修饰符从而查找不可分割的匹配

```json
GET luo/_search
{
  "query": {
    "match": {
      "address.keyword": "132 Gunnison Court"
    }
  }
}

// 因为是 keywrod，所以一旦稍微不一样，就不能查询到数据
GET luo/_search
{
  "query": {
    "match": {
      "address.keyword": "132 Gunnison" // 删掉一个单词
    }
  }
}
```



#### （8）高亮查询

```json
GET /test-db/_search
{
  "query": {
    "match": {
      "name": "罗俊华"
    }
  },
  "highlight": {
    "fields": {
      "name":{}
    }
  }
}
```



![Snip20201003_1](/Users/luo/Documents/开发笔记/images/Snip20201003_1.png)

使用`pre_tags`和`post_tags`自定义高亮标签

![Snip20201003_2](/Users/luo/Documents/开发笔记/images/Snip20201003_2.png)

### 6.批量操作bulk

> `{"index":{"_id":1}}`代表此次是要要索引文档进去
>
> `{"name":"罗俊华"}`代表此次要索引的数据
>
> `{"index":{"_id":2}}`代表第二个操作，依次类推
>
> 左边有几个操作右边的`items`中就有几个操作结果
>
> 每一个操作都是独立的，上一条操作失败，并不会影响下一条的操作，没有事务这一概念
>
> 只能在`kibana`下进行操作

![Snip20201004_10](/Users/luo/Documents/开发笔记/images/Snip20201004_10.png)

批量操作实例2

```json
POST /_bulk
{"delete":{"_index":"website","_type":"blog","_id":123}}
{"create":{"_index":"website","_type":"blog","_id":123}}
{"title":"我要插入一条数据"}
{"index":{"_index":"website","_type":"blog"}}
{"title":"我要插入第二条数据"}
{"update":{"_index":"website","_type":"blog","_id":123}}
{"doc":{"title":"我要更新id=123的数据"}}
```

![Snip20201004_11](/Users/luo/Documents/开发笔记/images/Snip20201004_11.png)

### 7.执行聚合 aggregategations

聚合提供了从数据中分组和提取数据的能力。最简单的聚合方法大致等于`SQL Group`，`By`和`SQL`聚合函数。在`elastic search`中，您有执行搜索返回`hits`（命中结果），并且同时返回聚合结果，把一个响应中的所有`hits`（命中结果）分隔开的能力。这是非常强大且有效的，您可以执行查询和多个聚合，并且在一次使用中得到各自的（任何一个的）返回结果，使用一次简介和简化`API`来避免网络往返

#### （1）使用`terms`聚合

1、搜索`address`中包含`mill`的所有人的年龄分布以及平均年龄

```json
GET luo/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAggregattion": {
      "terms": {
        "field": "age",
        "size": 10
      }
    }
  }
}
```

![Snip20201008_5](/Users/luo/Documents/开发笔记/images/Snip20201008_5.png)

> buckets中的 key代表的是age的已有值，`doc_count`指的是该值出现了几次

#### （2）聚合多个

```json
GET luo/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "aggs": {
    "ageAggregattion": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "age_avg_grade": { 
      "avg": {
        "field": "age" 
      } 
    },
    "balance_avg": { 
      "avg": {
        "field": "balance" 
      } 
    }

  }
}
```

![Snip20201008_7](/Users/luo/Documents/开发笔记/images/Snip20201008_7.png)

#### （3）子聚合

1、按照年龄聚合，并且请求这些年龄段的这些人的平均薪资

```json
GET luo/_search
{
  "aggs": {
    "ageAggregattion": {
      "terms": {
        "field": "age",
        "size": 10
      },
      "aggs": {
        "age_balance_avg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

2、查出所有年龄分布，并且这些年龄段中的M的平均薪资和F的平均薪资以及这个年龄段的总体平均薪资

```json

GET luo/_search
{
  "aggs": {
    "age_agg": {
      "terms": {
        "field": "age",
        "size": 10
      },
      "aggs": {
        "gender_agg": {
          "terms": {
            "field": "gender.keyword" //text 类型的字段使用聚合的时候需要装为 keyword类型
          },
          "aggs": {
            "age_balance_agg": {
              "avg": {
                "field": "balance"
              }
            }
          }
        },
        "balance_agg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
```

### 8.映射

**Elastic search 7 去掉type概念**

关系型数据库中两个数据表示是相对独立的，即使他们里面有相同名称的列也不影响使用，但是在`es`中却不是这样的。`es`是基于`Lucene`开发的搜索引擎，而`es`中不同的`type`下名称相同的`field`最终在`Lucene`中的处理方式是一样的。

* 两个不同`type`下的两个`user_name`，在es同一个索引下其实被认为是同一个`filed`，你必须在两个不同的`type`中定义相同的`filed`映射。否则，不同`type`中的相同字段就会在处理中出现冲突的情况，导致`Lucene`处理效率下降
* 去掉`type`就是为了提高`es`处理数据的效率

Elastic search 7.x

* URL中的`type`参数为可选，比如，索引一个文档不再要求提供文档类型

（1）获取索引的映射信息

```json
GET luo/_mapping
```



Elastic search 8.x

* 不再支持URL中的`type`参数

解决：将索引从多类型迁移到单类型，每种类型文档一个独立索引

### 9.数据迁移

`elastic search 6.0`及其以后版本

```json
POST _reindex
{
  "source": {
    "index": "luo"
  },
  "dest": {
    "index": "new-bank"
  }
}
```

老版本要指定老索引的类型

```json
POST _reindex
{
  "source": {
    "index": "luo",
    "type":"映射的名字" //指定索引的类型
  },
  "dest": {
    "index": "new-bank"
  }
}
```





## 六、spring boot操作es

### 1.官方api文档所在位置

![Snip20201003_3](/Users/luo/Documents/开发笔记/images/Snip20201003_3.png)

### 2.添加maven依赖，并更新版本

因为`spring boot dependencies`已经封装了`es`的版本，所以我们需要，手动更改`es`的版本

![Snip20201009_3](/Users/luo/Documents/开发笔记/images/Snip20201009_3.png)

```xml
    <properties>
        <elasticsearch.version>7.9.2</elasticsearch.version>

    </properties>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.elasticsearch.client/elasticsearch-rest-high-level-client -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>${elasticsearch.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.elasticsearch.client</groupId>
                    <artifactId>elasticsearch-rest-client</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.elasticsearch</groupId>
                    <artifactId>elasticsearch</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

<!--        这两个还是6.8.10，所以手动调节版本-->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-client</artifactId>
            <version>${elasticsearch.version}</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>${elasticsearch.version}</version>
        </dependency>
    </dependencies>
```



### 3.添加测试数据

> `https://github.com/elastic/elasticsearch/tree/master/docs/src/test/resources`下的`accounts.json`
>
> `POST /luo/_bulk`加上`accounts.json`中的内容

![Snip20201004_12](/Users/luo/Documents/开发笔记/images/Snip20201004_12.png)

### 4.选择客户端

#### （1）9300 TCP

`spring-data-elasticsearch:transport-api.jar`

* `spring boot`版本不同，`transport-api.jar`不同，不能适配`es`版本
* `es 7.x`已经不建议使用，8以后就要废弃

#### （2）9200 HTTP

`JestClient`：非官方，更新慢

`RestTemplate`：模拟发送`Http`请求，`Es`很多操作都要自己封装，麻烦

`HttpClient`：同上

`Elasticsearch-Rest-Client`：官方`RestClient`，封装了`es`操作，`API`层次分明，上手简单

### 5.es索引存储分析

#### （1）冗余存储

```json
{
  skuId:1,
  spuId:11,
  skuTitle:"华为手机",
  price:998,
  saleCount:99,
  attrs:[
  	{key:"尺寸",value:"5寸"},
		{key:"cpu",value:"高通"}.
		{key:"分辨率",value:"全高清"}
  ]
}
```

> 假设一个商品有20个`spu`属性，那么`100万`条数据将占用
>
> 100 0000 * 20KB = 2GB 内存
>
> 总结：空间（冗余）换时间

#### （2）数据分表，解耦

```json
// sku 索引
{
  skuId:1,
  spuId:11,
  skuTitle:"华为手机",
  price:998,
  saleCount:99
}

// spu索引
{
  spuId:11,
    attrs:[
  	{key:"尺寸",value:"5寸"},
		{key:"cpu",value:"高通"}.
		{key:"分辨率",value:"全高清"}
  ]
}
```

> 问题：`jd`上是根据现有的商品，通过`es`的聚合得出的商品标签，如果是固定的标签，就可能出现点击该标签，却没有对应商品的情况
>
> 假设：有10000个sku，其中总共涉及到了4000个spu，所以要分步查询
>
> 一、查询出这4000个spu对应的所有可能属性，
>
> 二、一次查询至少要携带[spuId,spuId,...]（一个有4000个spuId的数组作为查询条件）
>
> 4000 * 8b = 32kb   一次请求就要携带`32kb`的数据（spuId为long，所以是8b）
>
> 再假设此时有10000人在检索数据
>
> 32kb*10000 = 320mb 只是请求的数据就要占据这么多
>
> 总结：时间换空间



![Snip20201009_4](/Users/luo/Documents/开发笔记/images/Snip20201009_4.png)

### 6.创建索引

> nested https://www.elastic.co/guide/en/elasticsearch/reference/7.9/nested.html

```json
PUT product
{
  "mappings": {
    "properties": {
      "skuId":{
        "type": "long"
      },
      "spuId":{
        "type": "keyword"  //之后做折叠功能用到
      },
      "skuTitle":{
        "type": "text",
        "analyzer": "ik_smart"  // 指定分词器
      },
      "skuPrice":{
        "type": "double"			//keyword 做精确价格，不可用于范围查询
      },
      "skuImg":{
        "type": "keyword",
        "index": false,				// 不用图片URL来检索
        "doc_values": true   // 如果为true，这个属性可以用来做聚合操作（type也需要是keyword）
      },
      "saleCount":{
        "type": "long"
      },
      "hasStock":{
        "type": "boolean"
      },
      "hotScore":{
        "type": "long"
      },
      "brandId": {
        "type": "long"
      },
      "brandName": {
        "type": "keyword",   // keyword + doc_value 都是 true，才能进行聚合操作
        "index": false,
        "doc_values": true
      },
      "brandImg": {
        "type": "keyword",
        "index": false,
        "doc_values": true
      },
      "catelogId": {
        "type": "long"
      },
      "catelogName": {
        "type": "keyword",
        "index":false,
        "doc_values": true
      },
      "attrs":{
        "type": "nested",
        "properties": {
          "attrId":{
            "type":"long"
          },
          "attrName":{
            "type":"keyword",
            "index":false,
            "doc_values":true
          },
          "attrValue":{
            "type":"keyword"
          }
        }
      }
    }
  }
}
```

### 7.对象中的数组字段将会被扁平化处理

`elastic search`没有内部对象的概念，

```json
PUT my-index-000001/_doc/1
{
  "group" : "fans",
  "user" : [ 
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
```

所以内部对象会被转换为

```json
{
  "group" :        "fans",
  "user.first" : [ "alice", "john" ],
  "user.last" :  [ "smith", "white" ]
}
```

这样处理之后原本`user`的`first`和`last`之间的联系就被破坏了

```json
GET my-index-000001/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "user.first": "Alice" }},
        { "match": { "user.last":  "Smith" }}
      ]
    }
  }
}
```

原本查询不到的数据，经过扁平化处理后，查询到了错误的信息

![Snip20201009_5](/Users/luo/Documents/开发笔记/images/Snip20201009_5.png)

#### （1）解决

```json
// 创建索引的时候指定映射为 nested （嵌入），而不是 object 类型
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "user": {
        "type": "nested" 
      }
    }
  }
}

// 插入测试数据
PUT my-index-000001/_doc/1
{
  "group" : "fans",
  "user" : [
    {
      "first" : "John",
      "last" :  "Smith"
    },
    {
      "first" : "Alice",
      "last" :  "White"
    }
  ]
}
// 这样的查询就不再能查询到错误的数据了，因为 Alice Smith 没有在同一个 nested （嵌入）对象中
GET my-index-000001/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "Smith" }} 
          ]
        }
      }
    }
  }
}

// Alice White 在同一个 nested 对象中
GET my-index-000001/_search
{
  "query": {
    "nested": {
      "path": "user",
      "query": {
        "bool": {
          "must": [
            { "match": { "user.first": "Alice" }},
            { "match": { "user.last":  "White" }} 
          ]
        }
      },
      "inner_hits": {  // 允许高亮匹配到的 nested 文档
        "highlight": {
          "fields": {
            "user.first": {}
          }
        }
      }
    }
  }
}
```

## 七、搜索引擎思想的应用（根据销售属性的组合来反向搜索sku）

### 1.情况描述

每个`skuId`下有各自不相同的销售属性`attrValue`

现要通过多个`attrValue`的组合来反向检索出`skuId`

### 2.解决

创建`HashMap`，并以`attrValue`为`key`，以`List<Long>`（范型Long存放的是`spuId`）为`HashMap`的`value`，将有同一公有有属性`attrValue`的`spuId`放入到`List<Long>`集合中，再把`key-value-pair`放入`HashMap`中

这一操作类似搜索引擎的建立`倒排索引`

需要检索的时候，根据`attrValue`从`HashMap`中取出对应的`List<Long>`，就能判断出是哪几个`spuId`包含此`attrValue`，如果是多个`attrValue`来联合筛选`List<Long>`，求`交集`，就能筛选出符合条件的`spuId`

### 3.代码

#### （1）建立倒排索引

```java
    /**
     * 解析Sku 所拥有的 销售属性(saleAttr)，
     * @param skuInfoWithSaleEntitiesVoList
     * @return key：销售属性的value（attrValue），value：拥有该销售属性的 sku的id
     */
    private HashMap<String, SaleAttrAndSkuMapVo> parseSaleAttr(List<SkuInfoWithSaleEntitiesVo> skuInfoWithSaleEntitiesVoList) {


        HashMap<String, SaleAttrAndSkuMapVo> saleAttrAndSkuMap = new HashMap<>();

        skuInfoWithSaleEntitiesVoList.forEach(skuInfo->{

            if(!CollectionUtils.isEmpty(skuInfo.getSaleAttrList())){

                skuInfo.getSaleAttrList().forEach(saleAttr->{

                    // 判断map中有没有 key = attrValue 的对象
                    SaleAttrAndSkuMapVo saleAttrAndSkuMapVo = saleAttrAndSkuMap.get(saleAttr.getAttrValue());

                    if(ObjectUtils.isEmpty(saleAttrAndSkuMapVo)){

                        SaleAttrAndSkuMapVo saleAttrAndSkuMapVoNew = new SaleAttrAndSkuMapVo();

                        saleAttrAndSkuMapVoNew.setSaleAttrId(saleAttr.getId());

                        saleAttrAndSkuMapVoNew.setSkuIdListContainingSaleAttrValue(new ArrayList<>());

                        // 因为新增时，SaleAttrAndSkuMapVo 中不可能存在重复的 skuId，所以直接添加
                        saleAttrAndSkuMapVoNew.getSkuIdListContainingSaleAttrValue().add(skuInfo.getSkuId());

                        saleAttrAndSkuMap.put(saleAttr.getAttrValue(),saleAttrAndSkuMapVoNew);
                    }else{

                        // map 中已经存在 该 SaleAttrAndSkuMapVo ，因为 skuId不可能被添加两次，所以可以直接添加
                        saleAttrAndSkuMapVo.getSkuIdListContainingSaleAttrValue().add(skuInfo.getSkuId());

                    }

                });
            }

        });

        return saleAttrAndSkuMap;
    }
```

#### （2）检索

```javascript
            filterSku(attrValue) {
                //1、将不是 用户选中类型的 attrId

                let index = this.filterCondition.findIndex(f => {
                    return f === attrValue;
                })
                if (index === -1) {
                    this.filterCondition.push(attrValue);
                }

                console.log("条件数组：", this.filterCondition)

                if (this.item.saleAttrs.length === this.filterCondition.length) {

                    this.filterCondition.forEach((atttValue, index) => {

                        let satisfiedSkuIdList = this.item.saleAttrAndSkuMap[atttValue].skuIdListContainingSaleAttrValue.filter(skuId => {

                            if (index + 1 < this.filterCondition.length) {

                                let otherAttrValue = this.filterCondition[index + 1];

                                console.log("另一个attrValue是：", otherAttrValue)

                                let index2 = this.item.saleAttrAndSkuMap[otherAttrValue].skuIdListContainingSaleAttrValue.findIndex(otherSkuId => {

                                    return skuId === otherSkuId;

                                });

                                return index2 !== -1;
                            }
                        })

                        console.log("满足条件的skuId", satisfiedSkuIdList);

                        if(satisfiedSkuIdList.length>0){
                            //location.search = "?"
                            this.getSku(satisfiedSkuIdList[0]);
                        }

                    })

                } else {
                    console.log("只选中了一个条件")
                }
            }
```

