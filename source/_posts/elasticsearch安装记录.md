---
title: Elasticsearch + Kibana 安装记录
tags:
  - Kibana
  - Elasticsearch
categories: ELK
index_img: /img/kibana.png
date: 2020-04-05 12:47:32
---


公司今年的项目要把原来的数据迁移到Elasticsearch数据库中，通过技术负责人的介绍，我才知道了ES+kibana这个组合，开源，可以基于ES数据直接在Kibana上进行数据查询和图表分析。<!-- more --> 想我们之前还苦哈哈的自己写Echarts做数据图表，虽然实现了我们最初的需求，但是和采用Kibana比起来，还是占用了不少的时间和精力。

这几天在本地搭建了ES+Kibana，在此记录一下。

# Elasticsearch 安装

## 下载与安装

Elasticsearch的安装方式相对来说比较简单。`brew`的安装方式对我来说太慢，于是直接在官网上下载的安装包（公司网太慢，下个安装包还花了一天时间）。

- 下载地址：[https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)

- 安装版本：7.6.2 （与Kibana的版本要一致）

下载完成之后，放到`/usr/local/Cellar/`路径下，解压。

进入 bin 目录启动 ES 并在运行：

```
$ ./elasticsearch
$ ./elasticsearch -d (后台运行)
```

curl 测试是否正常运行（或者在浏览器中打开）：

```
$ curl 127.0.0.1:9200
```

此时出现：

```
{
  "name" : "mvQoSGm",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "4vUSt2_AQFSj5LZDVgR74g",
  "version" : {
    "number" : "7.6.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "04711c2",
    "build_date" : "2020-04-02T13:34:09.098244Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "6.6.0",
    "minimum_index_compatibility_version" : "6.0.0"
  },
  "tagline" : "You Know, for Search"
```

表示安装成功。

初步安装之后，我并没有做其他的设置，还有一些插件因为下载路径实在太慢，准备后面再慢慢安装。

## 查看ES集群的简单命令

### 1. 查看集群的健康状态

```
http://127.0.0.1:9200/_cat/health?v
```

URL中_cat表示查看信息，health表明返回的信息为集群健康信息，?v表示返回的信息加上头信息，跟返回JSON信息加上?。

- 集群的状态（status）：red红表示集群不可用，有故障。yellow黄表示集群不可靠但可用，一般单节点时就是此状态。green正常状态，表示集群一切正常。

- 节点数（node.total）：节点数，这里是2，表示该集群有两个节点。

- 数据节点数（node.data）：存储数据的节点数，这里是2。数据节点在Elasticsearch概念介绍有。

- 分片数（shards）：这是12，表示我们把数据分成多少块存储。

- 主分片数（pri）：primary shards，这里是6，实际上是分片数的两倍，因为有一个副本，如果有两个副本，这里的数量应该是分片数的三倍，这个会跟后面的索引分片数对应起来，这里只是个总数。

- 激活的分片百分比（active_shards_percent）：这里可以理解为加载的数据分片数，只有加载所有的分片数，集群才算正常启动，在启动的过程中，如果我们不断刷新这个页面，我们会发现这个百分比会不断加大。

![1](/img/1.png)

### 2. 查看集群的索引数

```
http://127.0.0.1:9200/_cat/indices?v
```

- 索引健康（health），green为正常，yellow表示索引不可靠（单节点），red索引不可用。与集群健康状态一致。

- 状态（status），表明索引是否打开。

- 索引名称（index），这里有.kibana和school。

- uuid，索引内部随机分配的名称，表示唯一标识这个索引。

- 主分片（pri），.kibana为1，school为5，加起来主分片数为6，这个就是集群的主分片数。

- 文档数（docs.count），school在之前的演示添加了两条记录，所以这里的文档数为2。

- 已删除文档数（docs.deleted），这里统计了被删除文档的数量。

- 索引存储的总容量（store.size），这里school索引的总容量为6.4kb，是主分片总容量的两倍，因为存在一个副本。

- 主分片的总容量（pri.store.size），这里school的主分片容量是7kb，是索引总容量的一半。

![2](/img/2.png)

### 3. 查看集群所在磁盘的分配状况

```
http://127.0.0.1:9200/_cat/allocation?v
```

返回集群中的各节点所在磁盘的磁盘状况。

### 4. 查看集群的节点

```
http://127.0.0.1:9200/_cat/nodes?v
```

通过该连接返回了集群中各节点的情况。这些信息中比较重要的是master列，带*星号表明该节点是主节点。带-表明该节点是从节点。

```
ip heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name

127.0.0.1 19 99 6 2.86 mdi * ruan-node-1

127.0.0.1 13 99 6 2.86 mdi - ruan-node-2
```

### 5. 查看集群的其它信息

```
http://127.0.0.1:9200/_cat/
```

获得查看集群信息的目录。

### 6. 全词搜索

```
http://127.0.0.1:9200/indexName/_search?pretty=true
```

`pretty=true` 表示格式化输出。


### 7. 精准搜索

```
http://127.0.0.1:9200/indexName/_search?q=123&pretty=true
```

表示搜索“123”。

### 8. 模糊搜索

```
http://127.0.0.1:9200/indexName/_search?q=*123*&pretty=true
```

模糊搜索“123”。

# Kibana 安装

## 下载与安装

Kibana安装时要注意与ES是同一个版本，ES我的版本是7.6.2，因此Kibana也下载的7.6.2版本的安装包。

- 下载地址同官网
- 版本：7.6.2

下载完成后，同样解压到`/usr/local/Cellar/`路径下。

进入kibana中的bin目录中，启动：

```
$ ./kibana
```

Kibana默认端口号*5601*， 启动成功后，到浏览器输入`hocalhost:5601`，就能进入Kibana页面了。

Kibana 7.x版本界面终于做的好看了一些。公司目前用的是5.x版本，界面简单粗暴，看起来总有些怪怪的。

![3](/img/kibana_cut.png)

**问题：**

在公司安装时，localhost:5601地址可以直接访问Kibana，但是回到家之后再次访问却提示localhost:5601错误，无法访问了，ES也只能通过127.0.0.1:9200才能访问，localhost：9200显示无法连接。

后来在`kibana.yml`配置文件里把localhost改成127.0.0.1之后，才能通过127.0.0.1:5601访问，不知道是不是我改动了一些配置？？

之后再调整一下看看。

## 使用

首先在`Management`中配置`Index Patterns`，将想要分析的Index加入进来。

在`Dev Tools`里写命令获取数据：

```
GET /data_index/_search
{
  "query": {
    "match_all": {}
  }
}
```

上面是最简单的获取，Kibana还有其他的Lucene查询语法，可以查询更多的ES数据，按条件查询、搜索等，待实际应用中慢慢学习。

今天先初步记录安装与使用的信息，以免过后忘记。

---

**参考来源：**

- *[ES查看集群信息命令](https://blog.csdn.net/genghaihua/article/details/81479619)*
- *[ElasticSearch常用查询命令](https://blog.csdn.net/a544258023/article/details/89709046?depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-2&utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-2)*