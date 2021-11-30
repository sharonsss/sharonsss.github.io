---
title: Elasticdump 踩坑记录
date: 2020-04-02 15:21:01
tags: Elasticsearch
categories: ELK
index_img: /img/zhishang2.jpg
---

> 感觉这几天是自己的智商低谷。😭

# Elasticdump 安装

为了解决远程ES数据库导入到本地ES的问题，今天在网上查了一天资料。了解到ELK中的Logstash可以实现这个需求，同时Logstash似乎还隐藏着更多其他炫酷功能，包括我后面可能要用到的web接口。<!-- more --> 感觉到这是一个稍微大点的功能，最终决定后面专门腾出时间来研究。

于是就找到了另外一个非常轻量级的ElasticSearch插件————Elasticdump，专门解决ES数据导入导出的问题。“dump”也是个非常形象的词了，有些简单粗暴，就跟它的实现一样。

安装Elasticdump很简单，mac上直接 `npm install elasticdump` 就好了。也可以全剧安装 `npm install elasticdump -g` 。

# 将远程 ES 数据导出到本地 JSON 文件

这里演示的是我把远程ES上的数据导出到本地的JSON文件。最初的想法是，把远程ES数据导到本地文件，再把文件导入本地的ES。此时，我并没有想到远程ES和本地ES可以直接导入导出，脑子卡在了本地文件里。😓

实际output部分为本地文件路径，具体如下：

```
# 格式：
$ elasticdump --input {protocol}://{host}:{port}/{index} --output ./test_index.json

# 例子：将ES中的test_index 中的索引导出
# 导出当前索引的mapping结构
$ elasticdump --input http://192.168.56.104:9200/test_index \
--output ./test_index_mapping.json --type=mapping

# 导出当前索引下的所有真实数据
$ elasticdump --input http://192.168.56.104:9200/test_index \
--output ./test_index.json --type=data
```

# 将远程的 ES 数据导出到本地 ES

本地JSON导出之后，终于想到elasticdump明明可以连接远程和本地ES。。。

## 无账号密码的情况下导出

如果远程和本地的ES都不需要账号密码访问权限，相对来说就比较容易，直接遵循下面的格式就可以。

官方提供的数据迁移示例：

```
# 拷贝analyzer分词
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=http://staging.es.com:9200/my_index \
  --type=analyzer
```

```
# 拷贝映射
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=http://staging.es.com:9200/my_index \
  --type=mapping
```

```
# 拷贝数据
elasticdump \
  --input=http://production.es.com:9200/my_index \
  --output=http://staging.es.com:9200/my_index \
  --type=data
```

## 设置账号密码登录导出

**据说 elasticdump 提供给了--httpAuthFile 参数来做认证**

```
--httpAuthFile      When using http auth provide credentials in ini file in form
                    `user=<username>
                    password=<password>`
```

只需要写一个ini文件 ，文件中写入用户名和密码就可以了，不过这个方法我还没有试。

另外一个好的方法是，**在--input参数和--output参数的的url中添加账号密码**。

例如：

```
$ elasticdump --input http://username:passowrd@production.es.com:9200/my_index \
--output http://username:password@staging.es.com:9200/my_index \--type=data
```

导出mapping

```
$ elasticdump --input http://username:password@host:9200/data_index \
--output ./test_index.json --type=mapping
```

导出所有data

```
$ elasticdump --input http://username:password@host:9200/data_index \
--output ./test_index_data.json --type=data
```

## 远程 ES 需要账号密码，本地不需要

**上面看起来行云流水一般就能搞定，事实上，对小白来说，好大的坑啊。。。**

折腾完上面的远程ES导出到本地JSON之外，突然发现可以直接从远程ES导入本地ES，为什么我还要拐个弯导出个本地文件，服了我自己的智商。。。

![智商](/img/zhishang1.jpg )

由于本地ES没有任何配置，于是我按照示例，最初想到了这样：

```
$ elasticdump --input http://username:password@0.0.0.0:9200/data_index \
--output http://localhost:9200/data_index --type=mapping
```

报错：

```
Error Emitted => {"root_cause":[{"type":"mapper_parsing_exception","reason":"Root mapping definition has unsupported parameters:  [kis_data_index_type : {properties={eng_exam_res={type=keyword}, item_state={type=keyword}, first_level={type=text}, chin_exam={analyzer=ik_max_word, type=text}, chinese_item={analyzer=ik_max_word, type=text}, xremark={type=keyword}, eng_synonym={type=keyword}, eng_exam={type=text}, contributor={type=text}, third_level={type=text}, chin_exam_res={type=keyword}, id={type=long}, remark_str={type=keyword}, chin_define_res={type=keyword}, chin_synonym={type=text}, eng_abbr={type=text}, contributor ={type=text, fields={keyword={ignore_above=256, type=keyword}}}, query={properties={match_all={type=object}}}, source_type={type=keyword}, english_item={analyzer=english, type=text}, second_level={type=text}, eng_define_res={type=keyword}, eng_define={type=text}, picture_res={type=keyword}, chin_abbr={type=text}, chin_define={type=text}}}]"}],"type":"mapper_parsing_exception","reason":"Root mapping definition has unsupported parameters:  [kis_data_index_type : {properties={eng_exam_res={type=keyword}, item_state={type=keyword}, first_level={type=text}, chin_exam={analyzer=ik_max_word, type=text}, chinese_item={analyzer=ik_max_word, type=text}, xremark={type=keyword}, eng_synonym={type=keyword}, eng_exam={type=text}, contributor={type=text}, third_level={type=text}, chin_exam_res={type=keyword}, id={type=long}, remark_str={type=keyword}, chin_define_res={type=keyword}, chin_synonym={type=text}, eng_abbr={type=text}, contributor ={type=text, fields={keyword={ignore_above=256, type=keyword}}}, query={properties={match_all={type=object}}}, source_type={type=keyword}, english_item={analyzer=english, type=text}, second_level={type=text}, eng_define_res={type=keyword}, eng_define={type=text}, picture_res={type=keyword}, chin_abbr={type=text}, chin_define={type=text}}}]"}
Error Emitted => {"root_cause":[{"type":"mapper_parsing_exception","reason":"Root mapping definition has unsupported parameters:  [kis_data_index_type : {properties={eng_exam_res={type=keyword}, item_state={type=keyword}, first_level={type=text}, chin_exam={analyzer=ik_max_word, type=text}, chinese_item={analyzer=ik_max_word, type=text}, xremark={type=keyword}, eng_synonym={type=keyword}, eng_exam={type=text}, contributor={type=text}, third_level={type=text}, chin_exam_res={type=keyword}, id={type=long}, remark_str={type=keyword}, chin_define_res={type=keyword}, chin_synonym={type=text}, eng_abbr={type=text}, contributor ={type=text, fields={keyword={ignore_above=256, type=keyword}}}, query={properties={match_all={type=object}}}, source_type={type=keyword}, english_item={analyzer=english, type=text}, second_level={type=text}, eng_define_res={type=keyword}, eng_define={type=text}, picture_res={type=keyword}, chin_abbr={type=text}, chin_define={type=text}}}]"}],"type":"mapper_parsing_exception","reason":"Root mapping definition has unsupported parameters:  [kis_data_index_type : {properties={eng_exam_res={type=keyword}, item_state={type=keyword}, first_level={type=text}, chin_exam={analyzer=ik_max_word, type=text}, chinese_item={analyzer=ik_max_word, type=text}, xremark={type=keyword}, eng_synonym={type=keyword}, eng_exam={type=text}, contributor={type=text}, third_level={type=text}, chin_exam_res={type=keyword}, id={type=long}, remark_str={type=keyword}, chin_define_res={type=keyword}, chin_synonym={type=text}, eng_abbr={type=text}, contributor ={type=text, fields={keyword={ignore_above=256, type=keyword}}}, query={properties={match_all={type=object}}}, source_type={type=keyword}, english_item={analyzer=english, type=text}, second_level={type=text}, eng_define_res={type=keyword}, eng_define={type=text}, picture_res={type=keyword}, chin_abbr={type=text}, chin_define={type=text}}}]"}
```

应该是不能直接用localhost。

于是我又试了这样：

```
$ elasticdump --input http://username:password@0.0.0.0:9200/kis_data_index \
--output http://127.0.0.1:9200/data_index --type=mapping
```

本机地址，没错吧？

依然报同样的错误。

于是，我寻思着是不是应该用电脑无线的IP？(我还加了公司无线网的账号密码。。)

```
$ elasticdump --input http://username:password@0.0.0.0:9200/data_index \
--output http://OA:password@10.9.1.000:9200/data_index --type=mapping
```

这次报错是连不上：

```
Thu, 02 Apr 2020 08:01:54 GMT | starting dump
Thu, 02 Apr 2020 08:01:54 GMT | got 1 objects from source elasticsearch (offset: 0)
Thu, 02 Apr 2020 08:02:14 GMT | Error Emitted => connect ECONNREFUSED 0.0.0.0:9200
Thu, 02 Apr 2020 08:02:14 GMT | Error Emitted => connect ECONNREFUSED 0.0.0.0:9200
Thu, 02 Apr 2020 08:02:14 GMT | Total Writes: 0
Thu, 02 Apr 2020 08:02:14 GMT | dump ended with error (get phase) => Error: connect ECONNREFUSED 0.0.0.0:9200:9200
```

把账号密码去了，还是提示连不上。

这时候我才看到指令最后的 --type=mapping。

不会是这个指令多余了吧？？

删掉之后，果然通了。。

```
$ elasticdump --input http://username:password@0.0.0.0:9200/data_index \
--output http://127.0.0.1:9200/data_index
```

**妈蛋!**
**被自己气吐血！！**

```
Thu, 02 Apr 2020 08:24:37 GMT | starting dump
Thu, 02 Apr 2020 08:24:37 GMT | got 100 objects from source elasticsearch (offset: 0)
Thu, 02 Apr 2020 08:24:38 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:38 GMT | got 100 objects from source elasticsearch (offset: 100)
Thu, 02 Apr 2020 08:24:38 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:38 GMT | got 100 objects from source elasticsearch (offset: 200)
Thu, 02 Apr 2020 08:24:38 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:38 GMT | got 100 objects from source elasticsearch (offset: 300)
Thu, 02 Apr 2020 08:24:38 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:38 GMT | got 100 objects from source elasticsearch (offset: 400)
Thu, 02 Apr 2020 08:24:38 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:38 GMT | got 100 objects from source elasticsearch (offset: 500)
Thu, 02 Apr 2020 08:24:38 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:42 GMT | got 100 objects from source elasticsearch (offset: 600)
Thu, 02 Apr 2020 08:24:42 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:42 GMT | got 100 objects from source elasticsearch (offset: 700)
Thu, 02 Apr 2020 08:24:43 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:43 GMT | got 100 objects from source elasticsearch (offset: 800)
Thu, 02 Apr 2020 08:24:43 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:43 GMT | got 100 objects from source elasticsearch (offset: 900)
Thu, 02 Apr 2020 08:24:43 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:43 GMT | got 100 objects from source elasticsearch (offset: 1000)
Thu, 02 Apr 2020 08:24:43 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:48 GMT | got 100 objects from source elasticsearch (offset: 1100)
Thu, 02 Apr 2020 08:24:48 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:48 GMT | got 100 objects from source elasticsearch (offset: 1200)
Thu, 02 Apr 2020 08:24:48 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:48 GMT | got 100 objects from source elasticsearch (offset: 1300)
Thu, 02 Apr 2020 08:24:48 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:48 GMT | got 100 objects from source elasticsearch (offset: 1400)
Thu, 02 Apr 2020 08:24:48 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:48 GMT | got 100 objects from source elasticsearch (offset: 1500)
Thu, 02 Apr 2020 08:24:48 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:52 GMT | got 100 objects from source elasticsearch (offset: 1600)
Thu, 02 Apr 2020 08:24:52 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:53 GMT | got 100 objects from source elasticsearch (offset: 1700)
Thu, 02 Apr 2020 08:24:53 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:53 GMT | got 100 objects from source elasticsearch (offset: 1800)
Thu, 02 Apr 2020 08:24:53 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:53 GMT | got 100 objects from source elasticsearch (offset: 1900)
Thu, 02 Apr 2020 08:24:53 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:24:53 GMT | got 100 objects from source elasticsearch (offset: 2000)
Thu, 02 Apr 2020 08:24:53 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:08 GMT | got 100 objects from source elasticsearch (offset: 2100)
Thu, 02 Apr 2020 08:25:08 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:08 GMT | got 100 objects from source elasticsearch (offset: 2200)
Thu, 02 Apr 2020 08:25:08 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:08 GMT | got 100 objects from source elasticsearch (offset: 2300)
Thu, 02 Apr 2020 08:25:08 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:08 GMT | got 100 objects from source elasticsearch (offset: 2400)
Thu, 02 Apr 2020 08:25:08 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:08 GMT | got 100 objects from source elasticsearch (offset: 2500)
Thu, 02 Apr 2020 08:25:08 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:08 GMT | got 100 objects from source elasticsearch (offset: 2600)
Thu, 02 Apr 2020 08:25:08 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:13 GMT | got 100 objects from source elasticsearch (offset: 2700)
Thu, 02 Apr 2020 08:25:13 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:13 GMT | got 100 objects from source elasticsearch (offset: 2800)
Thu, 02 Apr 2020 08:25:13 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:13 GMT | got 100 objects from source elasticsearch (offset: 2900)
Thu, 02 Apr 2020 08:25:13 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:13 GMT | got 100 objects from source elasticsearch (offset: 3000)
Thu, 02 Apr 2020 08:25:13 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:13 GMT | got 100 objects from source elasticsearch (offset: 3100)
Thu, 02 Apr 2020 08:25:13 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:18 GMT | got 100 objects from source elasticsearch (offset: 3200)
Thu, 02 Apr 2020 08:25:18 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:18 GMT | got 100 objects from source elasticsearch (offset: 3300)
Thu, 02 Apr 2020 08:25:18 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:18 GMT | got 100 objects from source elasticsearch (offset: 3400)
Thu, 02 Apr 2020 08:25:18 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:18 GMT | got 100 objects from source elasticsearch (offset: 3500)
Thu, 02 Apr 2020 08:25:18 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:18 GMT | got 100 objects from source elasticsearch (offset: 3600)
Thu, 02 Apr 2020 08:25:18 GMT | sent 100 objects to destination elasticsearch, wrote 100
Thu, 02 Apr 2020 08:25:23 GMT | got 35 objects from source elasticsearch (offset: 3700)
Thu, 02 Apr 2020 08:25:23 GMT | sent 35 objects to destination elasticsearch, wrote 35
Thu, 02 Apr 2020 08:25:23 GMT | got 0 objects from source elasticsearch (offset: 3735)
Thu, 02 Apr 2020 08:25:23 GMT | Total Writes: 3735
Thu, 02 Apr 2020 08:25:23 GMT | dump complete
```

不过，我在网上研究的时候，看到示例中都有指定--type=XXX，mapping和data要分别指定。后边需要验证一下不指定--type的情况下，导入到本地ES的文件是不是同时包含了mapping和所有data数据（从导入的记录上看，确实完整导入了3735条数据）。

---

**2020-04-03 补充：**

后来想到，--type无法访问可能是因为index指定的问题，后边遇到搜索查询数据的时候，再研究一下。
