---
title: 【Vue 学习笔记】Vue.js 实现在线翻译小项目
tags:
  - Vue
  - 百度翻译API
categories: 前端开发
index_img: /img/final-translate.png
date: 2020-05-30 16:17:13
---

> 用 Vue.js 实现一个在线翻译的小项目。学习课程用半天，课后总结居然用了一天？

<!-- more -->

# 前言

利用 Vue 的脚手架工具（CLI）搭建好一套开发环境之后，我跟着一个网上课程学习了如何开发一个简单的在线翻译小项目。

需要用到的知识点包括：

- 使用 `@vue/cli` 创建项目
- 创建多个组件
- 组件之间的调用
- `http` 访问请求
- 获取在线翻译 API key
- 应用 Bootstrap 主题

# 一、使用 @vue/cli 创建新项目

如何使用 `@vue/cli` 创建项目，在上一篇文章里已经记录过，这里就不再重复了。这次创建的项目还是在原来的 `Projects` 文件夹下，项目名称为 `my-project`。

需要注意的是，因为上一篇文章中，我已经创建了一个名为 `vueapp` 的项目，其默认端口为 `8080`，因此在新的项目创建好之后，需要改变默认的端口号，比如改为 `8081`。

端口号需要在配置文件里更改，也就是 `/config/inedx.js`这个文件：

```Javascript
// Various Dev Server settings
host: 'localhost', // can be overwritten by process.env.HOST
port: 8081, // can be overwritten by process.env.PORT, if port is in use, a free one will be determined
autoOpenBrowser: false,
errorOverlay: true,
notifyOnErrors: true,
poll: false, // https://webpack.js.org/configuration/dev-server/#devserver-watchoptions-
```

将 `port` 后面的的值改为`8081`，或者任意一个不被占用的端口号。

配置好之后，在浏览器访问 `http://localhost:8081`，就能看到 Vue 项目的初始页面了。

![init](/img/vue_init.png)

# 二、创建组件

将原模版中默认的内容删掉，重新创建我们需要的内容。

此次想要创建的在线翻译项目，包括三个部分：

1. 主组件（App.vue），用来显示页面标题
2. Form 组件（TranslateForm.vue），用来创建翻译词条输入，语言筛选框，以及搜索按钮
3. Ouutput 组件（TranslateOutput.vue），用来展示在线翻译的结果

## 1. 创建组件并引入

话不多说，先在 `/components` 目录下创建两个组件 `TranslateForm.vue` 和 `TranslateOutput.vue`。并填充好每个组件中的基本代码结构。

以 `TranslateForm.vue` 为例：

```html
<template>
  <div id="translateForm"></div>
</template>

<script>
  // 输出 name 到 App.vue
  export default {
    name: "translateForm",
  };
</script>

<style></style>
```

在这个组件中，定义好 `translateForm` 的 `id` 和要输出的 `name`，这样就可以在 `App.vue` 中引入， `TranslateOutput.vue` 组件同理。如果是小的组件，CSS 相关的的代码可以直接写到 `<style>` 标签中。

组件创建好之后，就可以在 `App.vue` 中引入两个组件：

```html
<template>
  <div id="app">
    <!-- 引入两个组件 -->
    <translateForm></translateForm>
    <translateOutput></translateOutput>
  </div>
</template>

<script>
  // import components
  import TranslateForm from "./components/TranslateForm";
  import TranslateOutput from "./components/TranslateOutput";

  export default {
    name: "App",
    // import components
    components: {
      TranslateForm,
      TranslateOutput,
    },
  };
</script>

<style>
  #app {
    font-family: "Avenir", Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
```

引入的位置有三个，一个在 `<template>` 中，一个在 `import` 中，一个在 `export default` 中的 `components` 中。

**需要注意的是，两个位置引入的组件名称是两个 `.vue` 文件的名称，并非组件里定义的 `name`。在这里，`.vue` 文件的名称首字母都是大写，而 `name` 中的第一个首字母是小写，注意区分。**

## 2. 填充页面的基本内容

接下来，我们先填充主页面的基本内容。在 `App.vue` 中，填充页面的标题和副标题：

```html
<template>
  <div id="app">
    <h1>在线翻译</h1>
    <h5>简单 / 易用 / 便捷</h5>
    <!-- 引入两个组件 -->
    <translateForm></translateForm>
    <translateOutput></translateOutput>
  </div>
</template>
```

在 `TranslateForm.vue` 中填充输入框、筛选框和“翻译”按钮：

```html
<template>
  <div id="translateForm">
    <div>
      <form>
        <input type="text" placeholder="输入需要翻译的内容" />
        <select>
          <option value="en">English</option>
          <option value="ru">Russian</option>
          <option value="kor">Korean</option>
          <option value="jp">Japanese</option>
        </select>
        <input type="submit" value="翻译" />
      </form>
    </div>
  </div>
</template>
```

此时，就可以在页面中看到填充的基本信息：

<img src="/img/基本页面.png" width="30%">

## 3. 定义事件

接下来，我们需要定义几个事件。

首先，需要获取用户在 `<input>` 标签中输入的值，在点击“翻译”按钮时，先 `alert` 一下输入的值。

分步来做：

1. 使“翻译”按钮生效
2. 获取 `<input>` 标签的值，并 `alert` 出来

给 `<form>` 标签定义一个点击提交事件 `v-on:submit="formSubmit"`，然后在 JS 部分中定义一个 `method`方法，实现`formSubmit` 的功能，此时点击“翻译”按钮，页面会弹出 `alert` 定义的文字 “hello world”。

```html
<template>
  <div id="translateForm">
    <div>
      <form v-on:submit="formSubmit">
        <input type="text" placeholder="输入需要翻译的内容" />
        <select v-model="language">
          <option value="en">English</option>
          <option value="ru">Russian</option>
          <option value="kor">Korean</option>
          <option value="jp">Japanese</option>
        </select>
        <input type="submit" value="翻译" />
      </form>
    </div>
  </div>
</template>

<script>
  export default {
    name: "translateForm",
    methods: {
      formSubmit: function () {
        alert("hello world");
      },
    },
  };
</script>
```

接下来，需要获取 `<input>` 标签的值，并 `alert` 出来，此时涉及到数据绑定。在 JS 中添加`data`方法，在这个方法中定义一个属性，然后使用 `v-model` 将`<input>` 的值绑定到这个属性上。

这个属性我命名为 `textToTranslate`，默认给一个空值：

```html
<script>
  export default {
    name: 'translateForm',
    data: function() {
        return {
            // 定义绑定到 <input> 标签中的属性
            textToTranslate: ""
        }
    }，
    methods: {
        formSubmit:function(){
            alert("hello world");
        }
    }
  }
</script>
```

之后，将 `textToTranslate` 属性绑定到 `<input>` 标签中。这时，当用户输入内容，并点击“翻译”按钮时，就可以在 `method` 方法中 `alert` 出用户输入的内容，也就是传递到 `textToTranslate` 中的值：

```html
<input type="text" v-model="textToTranslate" placeholder="输入需要翻译的内容" />
```

```javascript
methods: {
      formSubmit:function(){
          alert(this.textToTranslate);
      }
  }
```

这个时候，页面会有一次自动刷新，需要把页面自动刷新关掉（取消默认事件）：

```javascript
methods: {
      formSubmit:function(e){
          alert(this.textToTranslate);
          // 取消默认事件
          e.preventDefault();
      }
  }
```

## 4. 传递输入内容至根组件

`TranslateForm.vue` 组件中获取到的输入内容，**需要传递到根组件 `App.vue`，在根组件中将输入内容进行翻译，再将翻译结果传递给 `TranslateOutput.vue` 组件，然后显示出来。**

那么，怎么样才能把 `TranslateForm.vue` 组件中的内容传递给根组件（父组件）？

Vue 中提供了一个“事件注册”的方法，实现子组件向父组件传值：

```javascript
//第一个参数是注册事件的名称，名字随便起，第二参数是要传的值
this.$emit("事件名称", "要传递的值");
```

`TranslateForm.vue` 组件中，在 `method` 方法里注册一个事件 `formSubmitted`，传递的值就是 `this.textToTranslate`：

```javascript
methods: {
     formSubmit:function(e){
         // alert(this.textToTranslate);
         // 注册事件，传递输入的值给根组件
         this.$emit("formSubmitted", this.textToTranslate);
         // 取消默认事件
         e.preventDefault();
     }
 }
```

定义好之后，在 `App.vue` 中的相应位置来接收，将接收的事件定义为 `translateText`：

```html
<translateForm v-on:formSubmitted="translateText"></translateForm>

...

<script>
  methods: {
      // 定义一个参数 text，来接收子组件传递过来的值
      translateText: function(text) {
          alert(text);
      }
  }
</script>
```

这样，组件之间的传值问题就解决了。

## 5. 调用百度翻译 API 实现翻译

传递到跟组件的值，将借助百度翻译 API 来进行翻译。目前百度翻译提供免费的通用型 API，普通开发者可以直接申请 API key，调用限制为 1s 一次，对于这个小项目够用了。

**通用翻译 API HTTP 地址：**

- http://api.fanyi.baidu.com/api/trans/vip/translate

在此链接基础上，需要输入多个参数构成 API 的访问地址。为了保证调用安全，百度翻译 API 采用了生成签名的方式（md5 加密），具体生成方法在官方文档中有详细的举例。

![api](/img/baiduapi.png)

Javascript 中实现 md5 加密需要借助插件 `js-md5`：

```terminal
npm install --save js-md5

```

下载之后，需要在 `main.js` 文件中引入：

```javascript
import md5 from "js-md5";

Vue.prototype.$md5 = md5;
```

然后在 `App.vue` 中使用：

```javascript
// 按官方文档中的 step 1 生成拼接字符串，并存储到变量中
var baiduApi = xxxxxx;
// 将拼接字符串进行 md5 加密
var md5 = this.$md5(baiduApi);
```

## 6. `$http` 方法与 `$jsonp` 方法

在 Vue 项目中调用 API，可以使用 `http` 方式调用，也可以使用 `jsonp`，我们先使用前者。使用 `http` 调用，需要从项目所在目录安装 `vue-resource`:

```
npm install vue-resource --save
```

安装好之后，在 `main.js` 中引入：

```
import VueResource from 'vue-resource'
// 使用中间件
Vue.use(VueResource)
```

引入成功，回到 `App.vue` ，将 method 方法中的代码做相应改变，使用 `http` 请求，然后返回请求结果：

```javascript
<script>
methods: {
    // 定义一个参数 text，来接收子组件传递过来的值
    translateText: function(text) {
        // alert(text);
        this.$http.get('http://api.fanyi.baidu.com/api/trans/vip/translate?q=' + text + '&from=zh&to=en&appid=xxxx&salt=1435660288&sign=' + md5)
            // 返回请求结果 response
            .then((response)=>{
                // 打印请求结果
                console.log(response);
            })
    }
}
</script>
```

重新运行项目出现了 `No 'Access-Control-Allow-Origin'` 错误，产生了跨域的问题。我没有找到基于 `$http` 请求的跨域问题解决办法，于是又尝试了第二种方法：使用 `jsonp` 进行 API 请求。

- 同样，也需要先在项目目录下安装 `jsonp` 模块：`npm install vue-jsonp --save`；

- 在 `main.js` 中引入该模块

```
import VueJsonp from 'vue-jsonp'

// 使用中间件
Vue.use(VueJsonp)
```

引入之后，就可以在 `App.vue` 中使用 `jsonp` 进行跨域请求数据：

```javascript
var param = {
  header: {
    "content-type": "application/xml",
  },
};

// jsonp 解决跨域问题
this.$jsonp(
  "http://api.fanyi.baidu.com/api/trans/vip/translate?q=" +
    text +
    "&from=zh&to=en&appid=xxx&salt=1435660288&sign=" +
    md5,
  param
).then((response) => {
  console.log(response.trans_result[0].dst);
});
```

此时就可以获取到 JSON 数据了，我只需要翻译后的内容，也就是 `response.trans_result` 中的内容，根据 JSON 的结构来准确定位。

# 三、传递翻译结果到 `TranslateOutput.vue` 组件中

拿到翻译结果之后，接下来就需要传递给 `TranslateOutput.vue` 组件，在这个组件中显示。

在 `App.vue` 中写一个 `data` 方法，在这个方法中定义一个属性 `translatedText`，将其设置为空值，然后将 API 请求的结果存储到 `translatedText` 当中：

```javascript
data: function() {
    return {
      translatedText: ""
    }
  }

...

// jsonp 解决跨域问题
this.$jsonp('http://api.fanyi.baidu.com/api/trans/vip/translate?q=' + text
                + '&from=zh&to=en&appid=xxx&salt=1435660288&sign=' + md5, param)
    .then((response)=>{
    // console.log(response.trans_result[0].dst);
    // 将结果赋值给 translatedText
    this.translatedText = response.trans_result[0].dst
    })
```

并且在 html 的部分绑定这个属性：

```
<translateOutput v-text="translatedText"></translateOutput>
```

然后，在 `TranslateOutput.vue` 中实现 `translatedText` 这个属性：

```
export default {
  name: 'translateOutput',
  props: [
      "translatedText"
  ]
}
```

同时，需要在 html 的部分调用这个属性：

```
<template>
  <div id="translateOutput">
    <h2>{{translatedText}}</h2>
  </div>
</template>
```

到这里，在线翻译小项目的基本功能就实现的差不多了。

## 实现多语种翻译

上面的翻译结果都是基于英文的，要实现多语言的翻译，还需要在 `TranslateForm.vue` 组件里添加多个语言的选项：

```html
<select>
  <option value="en">English</option>
  <option value="ru">Russian</option>
  <option value="kor">Korean</option>
  <option value="jp">Japanese</option>
</select>
```

其中的 `value` 属性按照百度翻译 API 文档中的规定来写。

在 `<select>` 标签中定义一个名为 `language` 的属性，相应的属性在 `data` 方法中：

```
<select v-model="language">
<script>
data: function() {
      return {
          textToTranslate: "",
          language: ""
      }
  }
  </script>
```

`language` 属性定义好之后，需要将用户选择的 `language` 的值传递给根组件，以便在根组件的 API 请求中对应翻译的语种。实现方式同样借助 `$emit()` 注册事件，只需要在原来的注册事件中增加 `language` 的值：

```javascript
// 注册事件
this.$emit("formSubmitted", this.textToTranslate, this.language);
```

在 `App.vue` 中，用一个参数 `langage` 接收子组件传递过来的值，与前一个参数 `text` 是一样的逻辑。再将原 API 地址中的 `en` 替换为 `language`：

```javascript
methods: {
    translateText: function(text, language) {
        // API请求部分
    }
```

这样，多语言翻译就实现了。

不过，刷新页面之后，语言选项是空白，没有指定默认的选项。此时，可以通过添加 `created` 方法（该方法自动执行），来设置默认显示的选项：

```
export default {
  name: 'translateForm',
  ...
  created: function() {
      this.language = "en";
  }
}
```

# 四、应用 Bootstrap 主题更改页面样式

可以直接在 Bootswatch 这个网站上，下载想要的主题包，然后将包文件放到 `/static/css/` 的路径下，并在 `index.html` 文件中引入：

```
<link rel="stylesheet" href="/static/css/bootstrap.min.css">
```

更改根组件的一些样式，为副标题添加一个强调色：

```
<h5 class="text-muted">这是一个很小的 Vue 项目 😊</h5>
```

更改 `TranslateForm.vue` 组件的一些样式：

```html
<template>
  <div class="row" id="translateForm">
    <div class="offset-md-3 col-md-6">
      <form id="transForm" class="well form-inline" v-on:submit="formSubmit">
        <input
          class="form-control col-md-8"
          type="text"
          v-model="textToTranslate"
          placeholder="输入需要翻译的内容"
        />
        <select class="form-control col-md-2" v-model="language">
          <option value="en">English</option>
          <option value="ru">Russian</option>
          <option value="kor">Korean</option>
          <option value="jp">Japanese</option>
        </select>
        <input class="btn btn-primary  col-md-2" type="submit" value="翻译" />
      </form>
    </div>
  </div>
</template>
```

`offset-md-3 col-md-6` 这个属性是 Bootstrap 4 中的写法，和较早版本的写法稍有不同，需要注意一下。

还有一些样式，就直接在 `<style>` 标签里写了，比如添加边框，设置边框样式和颜色等等。

`TranslateOutput.vue` 组件中输入结果的字体不够明显，也可以直接在 `<style>` 标签里添加一些针对字体的样式。

**最终，这个在线翻译小项目的页面就是这个样子啦：**

![](/img/final-translate.png)

---

**参考来源：**

- _[百度翻译开放平台](http://api.fanyi.baidu.com/product/113)_
- _[js MD5 加密](https://blog.csdn.net/unbreakablec/article/details/91792652)_
- _[Vue.js 如何实现跨域请求？](https://www.zhihu.com/question/46202188)_
- _[vue 使用 jsonp 请求数据](https://www.cnblogs.com/wjw1014/p/11592444.html)_
- _[vue 中的 css 文件位置](https://www.csdn.net/gather_20/MtTaAg1sOTA2MS1ibG9n.html)_
- _[Bootswatch](https://bootswatch.com)_
