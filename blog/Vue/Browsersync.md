---
title: browsersync(浏览器同步工具)
date: {{ date }}
author: huangkai
tags:
    - Vue
---

配置使用 browsersync（）

## 一、介绍 ##
官网地址：
http://www.browsersync.cn/

Browsersync能让浏览器实时、快速响应您的文件更改（html、js、css、sass、less等）并自动刷新页面。更重要的是 Browsersync可以同时在PC、平板、手机等设备下进项调试。有了它，您不用在多个浏览器、多个设备间来回切换，频繁的刷新页面。更神奇的是您在一个浏览器中滚动页面、点击等行为也会同步到其他浏览器和设备中，这一切还可以通过可视化界面来控制。

### 1.1下载与安装：###


安装Node.js :  http://nodejs.cn/download/ 

安装 Browsersync :
- 全局安装: 打开终端执行 **npm install -g browser-sync**
- 在指定项目下安装:
进入指定项目的根目录，执行** npm install --save-dev browser-sync ** 或执行**npm i -D browser-sync** ，安装完成后，会在项目根目录下创建 node_modules文件目录并生成安装的文件，还会在根目录下的 package.json 文件中添加如下内容：
```
...
"devDependencies": {
    "browser-sync": "^2.23.6"
  }
...
```
上面 devDependencies 标签表示需要依赖的库，此文件中还有dependencies 标签也是依赖的库，两者区别就是 devDependencies 中所依赖的库并不是项目运行必须的，没有这些库项目也能运行起来，dependencies则是项目运行必须的库，不能少。

在 package.json文件中 scripts 标签添加如下内容:
```
"scripts": {
    "dev":"browser-sync start --server --files \"*.html,js/*.js,*.css",
    "start":"npm run dev"

  }
```

- dev：此名称可以随便定义 ，dev的值表示使用browser-sync启动，并监听根目录下的所有html 、js/*.js 、 *.css 的内容，如果有修改，浏览器将自动刷新。
- start ：是npm 固定的一个参数，值的意思就是使用npm 运行定义的 `dev` ,其实这个可以不定义，如果不定义，使用 npm 运行时dev时就必须加入 run 指令:语法为: `npm run dev`，如果定义了，就可以以 start 的方式启动，如 : `npm start` ，执行这个命令和执行 `npm run dev` 一样，也和 `npm run start` 一样，简单的说，使用start可以省略 `run` ， start就是dev的一个别名。 


browser-sync 其它参数请查看: http://www.browsersync.cn/docs/options/
