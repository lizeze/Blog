---
title: cache
date: 2019-09-05 10:02:17
tags:
---

# 前言
 都9102了，现在的前端如果还只是单纯的会写`HTML`、`JS`、`CSS`,那出去就不要说自己是前端开发了，说多了都是泪啊。
 
 最近前端面试中经常会被提到一个问题，那就是web的性能优化这一块，第一次被问到这个问题的时候瞬间脑子一片空白，不知道怎么回答。

之后就针对这一方面查了一些资料，当看到`缓存`这两个字的时候，这才恍然大悟，原来HTTP缓存是作为web性能优化的重要手段。下面就讲自己在网上学习之后的掌握的知识做一下整理。
# HTTP缓存
## 概述
缓存是一种保存资源副本并在下一次请求时直接使用副本的技术。当web缓存发现请求的资源已经被存储，它会拦截请求，而不会去资源服务器重新下载。这样带来的好处有：缓存服务器端压力，提升性能。对网站来说，缓存是达到高性能的重要组成部分。
## 缓存操作的目标
虽然HTTP缓存不是必须的，但重用缓存的资源通常是必要的。然而常见的HTTP缓存只能存贮`GET`响应，对应其他类型的响应则是无能为力。一般包括：
 * 一个`GET`请求，响应状态码为`200`,则表示成功。比如`HTML`文档，图片或者文件的响应
 * 永久重定向，状态码`301`
 * 错误响应: 响应状态码：404 的一个页面

## Cache-Control
HTTP/1.1定义的`Cache-Control`头用来区分缓存机制的支持情况，请求头和响应头都支持这个属性。

### 禁用缓存
缓存中不得存储客户端请求和服务器响应的各种文件，强制每次都要去资源服务器下载。
```
Cache-Control: no-store
```
### 缓存过期机制
在过期机制中主要用到的是`max-age=<seconds>`表示缓存能够使用的最大时间。是距离首次发起的时间(秒)。
```
Cache-Control: max-age=31536000
```

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba2ed7cdeb67ea?w=880&h=190&f=png&s=162660)
第一次请求

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba2f279a1ce6d5?w=2064&h=596&f=png&s=422107)
第二次请求

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba2f7c9cc81093?w=2066&h=640&f=png&s=447188)
![](https://user-gold-cdn.xitu.io/2019/6/29/16ba2f778c420a12?w=1344&h=504&f=png&s=370051)

>我们已经成功的做好了缓存，网页可以飕飕的运行了，但是我们可能会又想到，那如果在缓存还没有过期的时候我们发布了新的版本怎么办呢，那样客户不就没办法使用我们最新的版本了。下面马上解决这个问题.
### 更新缓存
之前每次发布版本之后我们都会提醒客户清除缓存、清除缓存、清除缓存!!!因为经常会有一些问题是因为没有清除缓存造成的，这个缓存可能是浏览器自行去缓存的，我们不太好控制，可是现在我们会了HTTP缓存，可以轻松的解决这个问题。下面看图

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba30446c6192dd?w=800&h=64&f=png&s=44564)
只需在文件加一个任意参数就可以，当有版本更新的时候只需更新文件后的参数就可以。

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba309b1b18d289?w=1666&h=194&f=png&s=223129)

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba30df2019a9fb?w=3410&h=530&f=png&s=256871)

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba30e62c2e30d1?w=1140&h=82&f=png&s=74072)

![](https://user-gold-cdn.xitu.io/2019/6/29/16ba30f94861abd4?w=3246&h=376&f=png&s=250522)
由上图可知

**HTTP缓存只是对相同链接的资源有效**
>示例中只有一个js文件，但是实际项目中可能会有很多的静态资源文件无法做到每次更新的时候修改对应的版本号。这个时候可以使用`webpack`中的`hash`码。每次打包之后都是不同的值，解压完美的解决这个问题。
## ETag
`HTTP`的`ETag`响应头是资源的特定版本标识符。可以让缓存更高效，并节省宽带，因为如果内容没有改变，web服务器不需要发送完整的响应。

如果给定URL中的资源更改，则一定要生成新的ETag值。

本次使用的ETag值使用的是`md5-file`

### 安装依赖

```
npm install --save md5-file
```
```
var data = fs.readFileSync('./js/main.js')
const md5File = require('md5-file')
 //使用工具获取到文件的md5值
    md5File('./js/main.js', (err, hash) => {
      if (err) throw err
      //将获取到的MD5值发送给响应头
      response.setHeader('ETag', hash)
      //获取请求头里是否包含标识符
      let ifNoneMatch = request.headers['if-none-match'];
      if (ifNoneMatch) {
        //如果标识符和当前的MD5不一致，则表示文件有更新，返回新文件
        if (ifNoneMatch !== hash) {
          response.statusCode = 200;
          response.write(data)
        } else {
          response.statusCode = 304;
        }
      } else {
        //如果不存在标识符则表示是第一次请求，直接返回文件
        response.write(data)
      }
      response.end()
    })

```

### 规则验证
![](https://user-gold-cdn.xitu.io/2019/6/29/16ba3f6673b3d034?w=1686&h=770&f=png&s=562829)

![](https://user-gold-cdn.xitu.io/2019/6/30/16ba3faf53d54575?w=2290&h=806&f=png&s=725942)

现在我们修改一下main.js文件

```
echo 'console.log(890)' >> js/main.js 
```

![](https://user-gold-cdn.xitu.io/2019/6/30/16ba428d2090f400?w=1348&h=842&f=png&s=634418)
 > 设置了`ETag`之后客户端还是会向服务器发送请求，只是返回的内容是不一样的。
 ## Expires
HTTP响应头信息(过期时间)。这个属性告诉缓存，相关缓存副本在什么时间内有有效的，过了这个时间就会向资源服务器发出请求。

```
Expires: Wed, 21 Oct 2015 07:28:00 GMT
```
`Expires`头设置的时间只能是必须是`格林威治时间（GMT）`

```
    let data = fs.readFileSync('./css/main.css')
    response.setHeader("Expires","Sun, 30 Jun 2019 02:51:26 GMT")
    response.write(data)
    response.end()
```

![](https://user-gold-cdn.xitu.io/2019/6/30/16ba64bb02f7fd42?w=1684&h=1476&f=png&s=1122646)
时间一秒一秒的过去了
![](https://user-gold-cdn.xitu.io/2019/6/30/16ba64f6b27c0142?w=1622&h=1252&f=png&s=913945)

就是这样，很简单的就实现了设置缓存的过期时间，但是有也一些要注意点地方：

* 因为过期时间是和客户端的时间去比较的，如果客户端的时间和服务器端的时间没有同步，那就就不会实现缓存的功能
* 时间设置的问题也不可以忽视，如果设置的时间是一个固定的时间，在下次更新版本的时候如果没有更新下次的过期时间，那么之后的请求也是会都发送的服务器的。
* 如果响应头中设置了`Cache-Control`,那么会优先使用`Cache-Control`
* 
> 设置了`Expires`之后如果副本文件没有过期，是不会在向服务器发送请求的
## Last-Modified

与 `Etag` 类似，`Last-Modified` HTTP 响应头也用来标识资源的有效性。 不同的是使用修改时间而不是实体标签。对应的请求头字段为`If-Modified-Since`

```
    let data = fs.readFileSync('./css/main.css')
    let stat = fs.statSync('./css/main.css')
    response.setHeader('Last-Modified', stat.mtime)
     if (request.headers['if-modified-since'] && stat.mtime == request.headers['if-modified-since']) {
      response.statusCode = 304;
      response.end()
      return
    }
    response.write(data)
    response.end()
```
发起第一次请求
![](https://user-gold-cdn.xitu.io/2019/6/30/16ba67ef96847f90?w=2014&h=678&f=png&s=443253)
发起第二次请求

![](https://user-gold-cdn.xitu.io/2019/6/30/16ba69b380fce8b0?w=1904&h=862&f=png&s=649455)
现在我们修改一下`main.css`文件

```
echo 'p{color:red;}' >> css/main.css 
```

![](https://user-gold-cdn.xitu.io/2019/6/30/16ba69f0e9587a01?w=2042&h=842&f=png&s=654906)

> 如果响应头头同时设置了`Cache-Control`和`Last-Modified`,那么会优先使用`Cache-Control`
 
 > 设置了`Last-Modified`之后，如果文件没有过期，客户端是不会向服务器发送请求