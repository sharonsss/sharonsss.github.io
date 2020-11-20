---
title: 【Vue 学习笔记】Vue 安装——使用命令行工具（CLI）脚手架
tags: Vue
categories: 前端开发
index_img: img/vue_init.png
date: 2020-05-25 11:35:12
---

> 学习 Vue 的时候，首先要关注的就是安装问题。今天要记录的是通过脚手架创建 Vue 新项目，搭建基本的开发环境。

<!-- more -->

## 前言

关于 Vue 的安装，官方文档中提供了几种方法。

如果是初学者，可以通过`npm`安装，或者直接在`<script>`中引入，具体可以参考官方的教程，简单明了。

这里我使用的是通过 Vue 的 CLI 脚手架工具来构建 Vue 的初始项目，搭建出基本的开发环境，自动配置好目录结构，可以方便的进行本地调试、单元测试、热加载（不重启项目，进行部分代码更新）以及本地部署等。

## Vue 脚手架工具安装

看网上安装教程的时候发现，之前的脚手架工具 `vue-cli` 已经更改为 **@vue/cli**，官方教程中的安装方式也做了一些调整。

如果通过之前的方式（`npm install -g vue-cli`）安装了 `vue-cli`，需要先进行卸载（`npm unistall vue-cli`），然后采用新的安装包名称`@vue/cli`：

```
进入项目目录
全局安装
npm install -g @vue/cli
```

回车，输入密码（密闻模式）。

此时会提醒`File exists...Remove the existing file and try again, or run npm with --force to overwrite files recklessly.`，是之前安装过 `vue-cli` 留下的一些文件。

如果不再需要，就强制覆盖掉：

```
npm install -g @vue/cli --force
```

安装成功，会显示 `@vue/cli` 以及一些依赖包的版本：

![screenshot](/img/vue0525.png)

<br>

## 基于 Webpack 模版 init 初始化项目

在项目目录下，创建一个基于 Webpack 模版的新项目。

之前的方式是执行命令：`vue init webpack projectname`，但是使用 `@vue/cli` 之后，需要先安装 `@vue/cli-init`，再进行初始化。

我创建的项目名称为 `vueapp`。

```
全局安装
npm install -g @vue/cli-init

再运行
vue init webpack vueapp
```

回车之后，会要求填写相应的信息：

```
? Project name vueapp
? Project description A Vue.js project
? Author sharonsss
? Vue build standalone
? Install vue-router? No
? Use ESLint to lint your code? No
? Set up unit tests No
? Setup e2e tests with Nightwatch? No
? Should we run `npm install` for you after the project has been created? (recommended) np
m

   vue-cli · Generated "vueapp".
```

初建项目的时候可以都默认，需要 install 的部分先选择不安装，后面再说。

最后一项会询问是否在创建了新项目之后自动安装依赖，默认选择“是”。不然也得自己手动执行安装。

```
# Installing project dependencies ...
# ========================

......

# Project initialization finished!
# ========================

To get started:

  cd vueapp
  npm run dev

Documentation can be found at https://vuejs-templates.github.io/webpack
```

依赖安装完成之后，会提示你如何启动项目。

执行 `npm run dev`，在浏览器访问 [http://localhost:8080](http://localhost:8080)，就能看到新项目初始化的页面啦。（8080 为默认端口，如果该端口被占用，可以修改为其他的端口）

也就是这个页面：

![ ](/img/vue_init.png)

此时，`vueapp` 项目路径下的目录是这个样子：

![folder](/img/vue_folder.png)

- 具体的项目开发，都是在 `src` 目录中进行的。

- `build`和`config`目录是和 webpack 配置相关的.

- `node_modules` 是我们通过 `npm install` 安装的依赖。

这样 `@vue/cli` 脚手架工具就成功安装了，接下来就可以在模版的基础上进行自己的 Vue 项目开发了。
