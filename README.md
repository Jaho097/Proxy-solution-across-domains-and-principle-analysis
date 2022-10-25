# vue3-vite-proxy-proxy-
vue3+vite用proxy解决跨域及proxy原理解析

---
theme: channing-cyan
---
一起养成写作习惯！这是我参与「掘金日新计划 · 4 月更文挑战」的第16天，[点击查看活动详情](https://juejin.cn/post/7080800226365145118 "https://juejin.cn/post/7080800226365145118")
### 前言
前端跨域的方式是面试八股文中常见的一部分，最近面试中也是被问到跨域的问题，跨域的解决方案无非就是几个
1. Jsonp跨域
2. cors
3. vue中的proxy跨域
本地项目中调试用的最多的就是 node代理
### 复习一下什么是跨域

> 浏览器从一个网页去请求另一个资源时，域名、端口、协议任一不同，都是跨域。在前后端分离的模式下，前后端的域名是不一致的，跨域是必然发生的事情。但服务器与服务器之间请求数据并不会存在跨域行为，跨域行为是浏览器安全策略限制，也称为浏览器的同源策略

如果是跨域，你将会发现下图的报错，经典的跨域问题，前端和后端都是我的本地项目，但仅仅是端口的不同，浏览器请求就发生了跨域的问题
```js
Access to XMLHttpRequest at 'http://127.0.0.1:8888/' from origin 'http://127.0.0.1:3000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

### 先开启一个后端node服务
由于需要用到一个后端接口来测试是否能够发生跨域效果，我们利用node开启一个服务，在vue项目之外开启一个node文件，然后执行以下代码
```js
//node1.js

var http = require("http");
http.createServer(function (request, response) {

    // 发送 HTTP 头部 
    // HTTP 状态值: 200 : OK
    // 内容类型: text/plain
    response.writeHead(200, {'Content-Type': 'text/plain'});

    // 发送响应数据 "Hello World"
    response.end('Hello World\n');
}).listen(8888);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8888/');
```
vscode中可以直接下载一个工具用来跑node文件，直接右键就能运行,非常的方便

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e489fdc35a5643d98091fee6b2b093e2~tplv-k3u1fbpfcp-watermark.image?)


![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb3c3b23663c4484896a79f8a881492f~tplv-k3u1fbpfcp-watermark.image?)

如果没有安装的同学可以执行命令 node node1.js 进行执行，服务开启后成功如图 

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a13e6f14bc8c48feb3bd21914818b356~tplv-k3u1fbpfcp-watermark.image?)


我们再在浏览器打开服务端口进行查看返回的结果测试一下

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/83fc95aa655b4b598cf239067727990d~tplv-k3u1fbpfcp-watermark.image?)


到这里我们的后端node服务已经能够启动了，那就说明可以当做一个接口来进行调用然后测试我们的跨域了




### 封装一个ajax

首先我们要发送请求，所以我们需要利用到ajax，这里引入axios太麻烦，所以直接用原生的ajax来发送请求，顺便学习一下如何手写一个原生的ajax请求
```js
function ajax(options) {
    //创建XMLHttpRequest对象
    const xhr = new XMLHttpRequest()


    //初始化参数的内容
    options = options || {}
    options.type = (options.type || 'GET').toUpperCase()
    options.dataType = options.dataType || 'json'
    const params = options.data

    //发送请求
    if (options.type === 'GET') {
        xhr.open('GET', options.url + '?' + params, true)
        xhr.send(null)
    } else if (options.type === 'POST') {
        xhr.open('POST', options.url, true)
        xhr.send(params)
    }
    //接收请求
    xhr.onreadystatechange = function () {
        if (xhr.readyState === 4) {
            let status = xhr.status
            if (status >= 200 && status < 300) {
                options.success && options.success(xhr.responseText, xhr.responseXML)
            } else {
                options.fail && options.fail(status)
            }
        }
    }
}
```
对我们的ajax请求进行一个简单的封装，这样就算是一个比较简单的手写的ajax，从request对象的创建到参数初始化，发送请求及接受相应都比较完整

### 发送一个ajax请求
在我们的vue项目中执行一个todo函数，放置我们请求的ajax函数，对应我们刚才启动的node服务进行请求，测试是否会发生跨域问题

```js
function todo(){
  ajax({
    type: 'post',
    dataType: 'json',
    data: {},
    url: 'http://127.0.0.1:8888',
    // url: '/api',
    success: function(text,xml){//请求成功后的回调函数
        console.log(text)
    },
    fail: function(status){////请求失败后的回调函数
        console.log(status)
    }
})
}
```
对函数执行后发现无可置疑，肯定是发生了跨域

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2f68547dbc046c6a8ab89e5fe9440fe~tplv-k3u1fbpfcp-watermark.image?)

由于端口的不同， vue项目跑在了3000端口，mode服务跑在了8888端口，所以发生了跨域的问题

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/102be6e419e3467b9505f9c304bcf639~tplv-k3u1fbpfcp-watermark.image?)

`这是配置代理跨域前的请求，记住这个格式待会和代理成功的做一个对比`

### 配置proxy
由于项目中使用的是vue3+vite包，所以我们打开vite.config.js

```js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import styleImport, { VantResolve } from 'vite-plugin-style-import';
// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    styleImport({
      resolves: [VantResolve()],
    }),],
    server: { //主要是加上这段代码
      host: '127.0.0.1',
      port: 3000,
      proxy: {
        '/api': {
          target: 'http://127.0.0.1:8888',	//实际请求地址
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        },
      }
    }
})

```
这个是代理成功的请求会是这个样子,我们发送的127.0.0.1:8888变成了127.0.0.1:3000,所以我们的代理生效了，不发跨域了，因为我们请求的和本地vue项目没有发送同源策略(请求地址和端口都是相同的)，`但是为什么我们请求成功了呢？`我们要请求的明明是8888端口才对呀？让我们看下面的解析
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a05f0fc22ff5424c899edfb5d93d3e79~tplv-k3u1fbpfcp-watermark.image?)<br>
从上图中，server是我们新增的部分，我们每一个配置都来解析一下
- host指的是将vue项目跑在哪个地址上
- prot则指的是vue项目跑的端口拉
如果不加这段配置的话，我们npm run dev的时候vue项目就不会跑在127.0.0.1:3000上，而是会自动分配一个地址和端口
-  proxy 其实就是利用了 Node 代理
-  target是我们实际要请求的地址
-  rewrite则是检测接口中出现的'api/'因为这段字符只是我们用来检测转发的而已，实际请求中要用正则将api设置为空，要记住他只是一个识别字符而已，因为我们的项目中可能存在需要请求不同的后端接口地址

意思就是识别到请求 ‘api/’ 就会变成 http://127.0.0.1:8888'

我们要调用的接口是 http://127.0.0.1:8888 <br>
但我们的项目是在127.0.0.1:3000所以proxy会做如下代理进行转发<br>
127.0.0.1:3000/api/ -> http://127.0.0.1:8888<br>
如果还有后续地址的话
127.0.0.1:3000/api/test/test1 -> http://127.0.0.1::8888/test/test1<br>

### 总结
proxy其实就是因为浏览器同源协议无法请求非同源的地址，但是服务器直接没有同源协议，利用将本地请求转到本地服务器进行代理转发，从而绕过了同源协议的限制，通过代理的实现可以解决跨域的问题，换了个思路去解决实际产生的问题，实在是一个比较好的解决方法

参考[文章](https://blog.csdn.net/weixin_43972437/article/details/107291071)
