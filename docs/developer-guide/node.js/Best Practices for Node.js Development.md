[TOC]

# 目录

## 开发指南

### 1) Node.js Version

新版的Node.js在性能上只会更优，可参考:  [benchmarking.nodejs.org](https://benchmarking.nodejs.org/)

### 2) Static Resource

静态资源让Nginx去管，主要是为了考虑灵活性，方便以后迁移到CDN。

### 3) Router

路由的基本原理实现:

```
class Express {
     private _stacks = [];
     use(middleware) {
         this._stacks.push(middleware);
     }
     private _routes = {};
     router(path , handle) {
         this._routes[path] = handle;
     }

     match(req , res) {
         for (path in this._routes) {
             if (path.match(req.url)) {
                 this._routes[path](req , res);
                 return;
             }
         }
     }

     exec(req , res) {
         var ind = 0;
         var self = this;
         function next() {
             ind+=1;
             self._stacks[ind] && self._stacks[ind](req , res , next);
         }
         this._stacks[ind](req , res , next);
     }
}

function express () {
     var nativeExpress = new Express();
     var server = http.createServer(function (request, response) {
             nativeExpress.exec(request , response);
             nativeExpress.match(request , response);
         });
     });
     server.listen(PORT);
     return nativeExpress;
}

module.exports = express;

var server = express();
server.router("/index.html" , function (req , res) {

})
server.use(function (req , res , next) {
     next();
})
```

针对静态路由规则，可以采用Map优化，选择框架的时候，一定要注意其是否针对路由匹配有做过相关优化

针对Restful动态路由，将[逐条匹配优化为单条匹配](http://www.symfonychina.com/blog/new-in-symfony-4-1-fastest-php-router)，效率应该能提升两个数量级; 或者采用基于**前缀树**的实现 [koajs/trie-router](https://github.com/koajs/trie-router) 优化路由匹配的次数。

### 4) JWT

不需要每来一个请求都要请求一次Redis，这一点在PHP 中更明显。无法常驻内存的程序最大的问题就是没有办法做到连接的复用。

### 5) Sync I/O

用异步尽量别用同步代码，毕竟Node.js是单线程的，同步代码会阻塞线程执行。比如: 文件操作，Gzip。

#### 5.1) Promise

推荐使用`Bluebird`实现的`Promise`

其中抛出的异常一定要catch，不然可能会造成程序异常退出

另外在理解Promise概念的时候，首先要明白Node.js里面定时器是个啥?

### 6) Parallel

降低时延

#### 6.1) Array + Promise

并发操控数组元素，实现类似Spark中并行处理的效果:

- [数组的遍历你都会用了，那Promise版本的呢](https://segmentfault.com/a/1190000014598785)

### 7) Stream

- [通过node的pipe实现请求代理](https://www.jianshu.com/p/1fce84c242f1)
- [文件转存服务](https://github.com/andycall/blog/issues/1)
- [Sending a streaming zip file in node.js](https://blog.championswimmer.in/2016/06/sending-a-streaming-zip-file-in-node-js/)
- [How to read file from ZIP using InputStream?](https://stackoverflow.com/questions/23869228/how-to-read-file-from-zip-using-inputstream)

### 8) Websocket

- [以中间件，路由，跨进程事件的姿势使用WebSocket](https://segmentfault.com/a/1190000016914790)

### 9) Template Engine

- 代码更易维护，写起来更加优雅
- 利用模板缓存，优化后端渲染

### 10) Gzip Compress

适当压缩会减少请求的网络时延，在弱网络环境下必须考虑。但Gzip相对而言占用较多CPU资源，阻塞主线程，所以压缩操作最好在其他进程或线程中去做。而Node.js单线程的特点决定了无法很好的利用多核CPU，所以最好是让Nginx去做Gzip压缩。

### 11) Middleware Policy

- [中间件执行模块koa-compose源码分析](https://segmentfault.com/a/1190000013447551)
- [Express中间件摆放顺序之坑](https://go.kieran.top/post/34/) 顺序很重要

### 12) Cluster Mode

[Node.js Cluster](http://nodejs.cn/api/cluster.html) 工作模式，常应用在后端截图、图片合成以及任务处理等场景中，不过需要注意的是，渲染引擎往往比较耗内存，所以不可能创建太多puppeteer/phantom实例。

- [evantahler/background_jobs_node](https://github.com/evantahler/background_jobs_node/blob/master/3-local.js)

另外需要留意Node.js父进程如何与子进程通信，以及两个子进程间如何通信？有没有类似Golang通道的概念？

Sails.js支不支持该模式？反正[Egg](https://eggjs.org/en/advanced/cluster-client.html) 支持

### 13) Module Development

模块化开发，公共模块以及各种业务模块

### 14) Context Validator

最好将validator抽离出来，别混在业务逻辑里面，如Sails的input参数校验，koa不知道有没有类似的插件

参考: [How to validate user input in a Node.js application](https://rethinkdb.com/blog/validation-techniques/)

### 15) Emit

观察者模式的实现，用于解耦程序，不过要注意`EventEmitter.emit`是同步方法呦。当然也可以借助MQ实现订阅发布。

Demo: [The beautiful thing called EventEmitter](https://dev.to/tunaxor/the-beautiful-thing-called-eventemitter-23ei)

### 16) 内存使用分析

哪些占用的空间较多？

### 17) 异常处理

## 异常汇总

### 1) The request header content contains invalid characters

`Url`中含有一些未转义的字符

### 2) Error: socket hang up

程序问题，看看有没有响应请求

### 3) 请求响应的数据乱码

十有八九是Gzip搞得鬼

### 4) 客户端请求并发限制

主要是[maxSockets](https://nodejs.org/api/http.html#http_new_agent_options)参数限制了客户端，在做客户端测试的时候记得配置该值。

```
// server.js
// 处理一个请求需要5秒，通过setTimeout设置5秒后响应
var http = require("http");
var n = 0;
http.createServer(function (req, res) {
    n++;
    setTimeout(function() {
        console.log("Accept " + n + "request.");
        res.end("test");  
    }, 5000);
}).listen(3000);

// client.js
exports.send = function () {
    var http = require('http');
    var options = {
        host: 'localhost',
        port: '3000',
        path: '/',
        method: 'GET',
    };

    var req = http.request(options, function(res){
        res.setEncoding('utf8');
        res.on('data', function (c) {
            //console.log(c);
        });
        res.on('end', function() {
            exports.success += 1;
            console.log("Response: " + exports.success);
        });
    });
    req.end();
};
exports.success = 0;

// attack.js
var client = require('./client');

var d = 1000,
    t = Date.now();
while(Date.now() - t < d) {
    client.send();
}
console.log('end.');
```

### 5) 跨域

了解跨域问题前先要知道一个概念: [preflight request](https://www.jianshu.com/p/b55086cbd9af)

然后再看如何应对此类问题，主要在请求头上下功夫:

请求头 | 含义 | 备注
---|---|----
`Access-Control-Allow-Origin` | 允许的请求域 | 可以配置为`*`
`Access-Control-Allow-Headers` |  允许的请求头 | 不能配置为`*`, Chrome中会报错
`Access-Control-Allow-Methods` | 允许的请求方法 | 特别注意处理`OPTIONS`

最终的代码看起来是这个样子：

```
app.all('*', function (req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Content-Type,Content-Length, Authorization, Accept,X-Requested-With");
    res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
    if (req.method === "OPTIONS") {
        return res.send(200); /*让options请求快速返回*/
    }
    next();
});
```

### 6) https协议访问

需要注意跳过签名认证: [Node.JS requests with rejectUnauthorized as false](https://stackoverflow.com/questions/36588963/node-js-requests-with-rejectunauthorized-as-false)

### 7) 获取客户端IP

- [获取客户端真实ip默认是ipv6格式](https://www.jianshu.com/p/bcab08f2f924)
- [获取req请求的原始IP](https://www.cnblogs.com/duhuo/p/5700900.html)

### 8) pm2对session的影响

用pm2启动node服务需要注意session信息不能保存在内存中，否则会出现**获取会话信息异常**的问题，原因就是一般情况下进程之间无法共享系统分配的资源

### 9) pm2失败重试次数过多

就需要停到之后再启动了。。。而不是重启就可以了事了。

### 10) 连接2min超时限制

### 11) OOM

构建编译时偶尔报错。。。需要调V8的配置`--max-old-space-size`了。

而程序问题引起的内存溢出往往是因为操作的数据量较大。。。要么优化业务，要么提高V8的配置。

可参考下面几篇文章:

- [【译】容器环境下Node.js的内存管理](https://juejin.im/post/5cef9efc6fb9a07ec56e5cc5)
- [Node.js内存溢出-process out of memory 问题的处理](https://itbilu.com/nodejs/core/Ey_SnYXnx.html)

### 12) node_redis不支持Redis Cluster

### 13) pm2对定时任务的影响

启动多个Node实例可能会产生同一事件被消费多次的问题

### 14) node-postgres Error: Connection terminated unexpectedly

这个问题遇到的人还挺多，但现在无解。只能做一些保护了:  [Connection Error crashing program #1449](https://github.com/brianc/node-postgres/issues/1449)

```
// 捕获回调抛出的异常
process.on('uncaughtException', function handleError (err) {
    logger.error("uncaughtException catched.****************************************");
    logger.error(err);
});
// 捕获Promise抛出的异常
process.on('unhandledRejection', function handleError (err) {
    logger.error("unhandledRejection catched.****************************************");
    logger.error(err);
}
});
```

### 15) pending...

## 常用命令

### 1) npm list -g --depth 0

查看全局安装过的npm包

### 2) pm2

首先要明确它能干啥: [Compare with pm2 and Forever](http://strong-pm.io/compare/)

```bash
# 安装
npm install pm2 -g

# 查询npm package
npm list pm2 -g

# 启动
pm2 start app.js -i 1 --name "api"

# 以fork模式启动(session可保存在内存中)
pm2 start app.js -x

# 配置内存的到达阈值自动重启

# node服务列表
pm2 list

# 实时监控
pm2 monit
pm2-gui

# 热更新
# update the code
pm2 reload all(重新耗时10s，占用时间较长)
pm2 start app.js --watch

# 关闭
pm2 kill
pm2 stop all
```

再看标配控件，可以考虑用一下pm2的日志插件 [pm2-logrotate](https://github.com/keymetrics/pm2-logrotate) 

### 3) npm prune --production

## 必读物

- [Node.js的10个性能优化技巧](https://www.jb51.net/article/52187.htm)
- [基于Node.js的苏宁电商核心业务实践](https://myslide.cn/slides/9796)
- [苏宁Node.js性能优化实战](https://mp.weixin.qq.com/s/JxRO5BhJai-tT6xWvFpKgQ)
- [4类 JavaScript 内存泄漏及如何避免](https://jinlong.github.io/2016/05/01/4-Types-of-Memory-Leaks-in-JavaScript-and-How-to-Get-Rid-Of-Them/)
- [How JavaScript works: memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)
- [JavaScript内存泄漏教程](http://www.ruanyifeng.com/blog/2017/04/memory-leak.html)
- [爬虫的广度优先和深度优先算法](https://www.cnblogs.com/wangshuyi/p/6734523.html)
- [i0natan/nodebestpractices](https://github.com/i0natan/nodebestpractices)
- [Node在有赞的实践](https://tech.youzan.com/youzan-node/) 中间层的工作(请求认证，页面渲染，请求转发，业务编排)，以及微服务化历程
- [Egg.js设计原则](https://eggjs.org/zh-cn/intro/index.html)
