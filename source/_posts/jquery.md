---
title: jQuery需要注意的几个小点
tags: jQuery
categories: 前端开发
index_img: /img/search.jpg
date: 2020-04-11 17:58:36
---


最近在写一个前端的搜索页面，主要思路是从Elasticsearch拿到数据，并在前端做搜索查询的极简页面。ES有提供现成的Javascript API，既有后端Node.js的版本，也有直接从client端就可以获取数据对的接口。<!-- more -->

我这次最开始用的是基于Client端直接获取数据的版本。（因为Node.js传参我还搞不太清楚。。）

前端的实现主要基于原生JavaScript和jQuery，记录下遇到的一些问题。

## 1. jQuery 引用链接的位置

jQuery在html文件中引用的`<script>`位置，要放到`<body>`标签之后，如果放到`<body>`标签之前，运行的时候会报错。

## 2. `<form action="#">` 元素

`<form>`标签提供了一个`action`的属性，这个属性的作用是，点击按钮会刷新页面。

我的页面中写了一个搜索按钮，按钮包含在`<form>`标签中，于是导致每次我点击“搜索”的时候，页面都会刷新，去掉这个属性就好。

## 3. 用 jQuery 判定输入的字符是中文还是英文

思路是用正则表达式来识别，同时利用jQuery的`.test()`方法。基本的正则表达式还要再复习一下。

判断字符是否为全英文：

``` Javascript
// 判断输入是否为全英文
var re=/^[a-zA-Z]+$/;

//判断是否包含数字字母下划线  当使用这个时如果只有部分是中文字符还可以使用英文字体
var reg=/[A-Za-z]*[a-z0-9_-]|\s$/;

if(!re.test(txt)){
    return false;
}
return true;
```

判断输入是否为全中文：

```Javascript
// 判断输入是否为全中文
var res=/^[\u4e00-\u9fa5]+$/;

if(!res.test(txt)){
    return false;
}
return true;
```

我涉及到的字符输入除了英文就是中文，还有少部分的数字，因此先判定英文字符，如果不是英文，那么就是中文或数字了，规则比较简单。

## 4. JS 里的 forEach() 和 jQuery 里的 each()

如果参数为一个，这一个参数代表“元素”。JS举例：

``` JavaScript
var arr = new Array(["b", 2, "a", 4],["c",3,"d",6]);
arr.forEach(function(item){
    alert(item);  //b, 2, a, 4和c,3,d,6
});
```

如果forEach里有两个参数，则第一个参数为该集合里的元素，第二个参数为集合的索引：

```Javascript
arr.forEach(function(item, i){

}
```

但是在jQuery中，如果至哟耦哦一个参数，那么这个参数代表“索引”。举例：

```Javascript
var arr = new Array(["b", 2, "a", 4],["c",3,"d",6]);
$.each(arr, function(item){
    alert(item);  //0;1
});
```

如果有两个参数，则第一个为索引，第二个该集合里的元素:

```Javascript
$.each(arr, function(i, item){

}
```

## 5. Bootstrap3-typeahead.js 输入框自动补全

这次做搜索页面，一个重要的功能是允许用户在搜索时，能出现模糊匹配的词汇（中文和英文），于是用了 bootstrap3-typeahead.js 这个插件。

typeahead的基本逻辑也很简单，通过访问数据库获取JSON数据之后，利用现成的`process()`函数就可以自动把获取到的数据显示在搜索框中。

需要注意的是获取的JSON格式，如果数据结构不符合，需要稍微做一些处理，再输入到`process`函数中。另外，下载的bootstrap3-typeahead.js 在引用时，注意要放到jQuery文件的后面，不然无法调用。

实现的效果也比较简单，如果有现成的数组，好说：

```Javascript
$(function () {
            var localArrayData = ['beijing', 'shanghai', 'guangzhou', 'tianjin', 'hangzhou', 'ningbo'];
            $("#txtUser").typeahead({
                source: localArrayData
            });
        });
```

这里的`source`，就是获取到的数据。

如果是需要调取数据库的数据，`typeahead()`方法中，要包含参数为`query, process`的函数，用来获取数据，并通过`process()`显示出来。

以ajax为例：

```Javascript
 $(function () {
            $("#txtUser").typeahead({
                source: function (query, process) {
                    $.ajax({
                        url: '/Tools/GetOperUsers',
                        data: {
                            name: query
                        },
                        type: 'post',
                        dataType: "json",
                        success: function (data) {
                            var res = [];
                            $.each(data, function (i, item) {
                                var aItem = { id: item.CreateUserId, name: item.CreateUserRealName };//把后台传回来的数据处理成带name形式
                                res.push(aItem);
                            })
                            // res 是我们获取到的数据
                            return process(res);
                        }
                    });
                }
            });
        });
```

ES 数据库的数据调取稍有不同，同样参照 client 端的 JS API。

我在前端实现之后是这样的：

![1](/img/typeahead.png)

效果很简单了。

后边想自定义调整一下显示栏的样式，如果实现了再补充。

## 6. 文本框刷新之后，清除原来的输入

这个点解决起来不要太简单!

只需要在`<input>`文本输入框中加一条属性`autocomplete="off"`，就好了。。

作用域不只是 `<input>` 标签，`<form>` 标签同样可以。
