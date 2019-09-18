---
title: phantomjs
date: 2019-09-18 22:53:37
tags:
---
# 概述
  有一个这样的需求，就是把页面上的`Echart`图表转换成图片保存到`word`中。
  
# 环境准备
  安装PhantomJS
  ```
  npm install phantomjs -g
  ```
  使用下面的命令，查看是否安装成功
  ```
  phantomjs --version
  ```
  # 实现
  ## webpage
   webpage模块是PhantomJS的核心模块，用于网页操作。
   ```javascript
  var webPage = require('webpage');
  var page = webPage.create();
   ```
 
# open()
open方法用于打开具体的网页。
```
var page = require('webpage').create();
page.open('http://127.0.0.1:8080/index.html', function (status) {
 page.render('google_home.png', { format: 'png' });
  phantom.exit();
});
```

# 测试
 执行
```
phantomjs  index.js
```

![](https://user-gold-cdn.xitu.io/2019/9/18/16d44d7ef2bf3192?w=1114&h=64&f=png&s=24861)

# 参数

```
var webPage = require('webpage');
var page = webPage.create();
var system = require('system');
var list = (system.args + "").split(',');
```


![](https://user-gold-cdn.xitu.io/2019/9/18/16d44da62d269ca4?w=1158&h=140&f=png&s=115317)

[Demo](https://github.com/lizeze/phantomjs-demo)
