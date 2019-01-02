[TOC]

## 参考指南

### 1. Node.js Version

新版的Node.js在性能上只会更优，可参考:  [benchmarking.nodejs.org](https://benchmarking.nodejs.org/)

### 2. Use built-in modules

尽量使用Node.js内置的模块，而不引入第三方库，当然此事要把握好一个度。

### 3. Static Resource

静态资源让Nginx去管，避免不必要的负载，也方便以后采用CDN

### 4. Router Cache

针对静态路由规则，可以采用Map优化，选择框架的时候，一定要注意其是否针对路由匹配有做过相关优化

针对Restful动态路由，将逐条匹配优化为单条匹配，效率应该能提升两个数量级

让单线程的Node.js做大量的正则匹配是很占用时间的

### 5. JWT -- Avoid Session

不需要每来一个请求都要请求一次Redis，这一点在PHP 中更明显。无法常驻内存的程序最大的问题就是没有办法做到连接的复用。

### 6. Sync I/O

用异步尽量别用同步代码，毕竟Node.js是单线程的，同步代码会阻塞线程执行。比如: 文件操作，Gzip。

### 7. Promise Performance

推荐使用`Bluebird`实现的`Promise`

其中抛出的异常一定要catch，不然可能会造成程序异常退出

### 8. Parallel Process

能并行绝对不串行，缩短响应时间，如果是在做爬虫，请注意别太嚣张，不然很容易触发对方的底线(屏蔽请求)

### 9. Stream

文件转存(非文件上传)，请求代理，建议用Stream

### 10. JSON Serialize

JSON序列化优化在Java/Go中很常见，在其他语言中其实也是个话题，类似的还有ProtoBuffer协议性能问题。

可采用 [fast-json-stringify](https://github.com/fastify/fast-json-stringify) 提升JSON解析的性能，但会增加编写的代码量，影响代码可读性。

### 11. Template Cache

针对模板，先考虑用缓存，然后再考虑重新拼接。尤其是做后端渲染的时候，一定要考虑。

### 12. Gzip Compress

适当压缩会减少请求的网络时延，在弱网络环境下必须考虑。但Gzip相对而言占用较多CPU资源，阻塞主线程，所以压缩操作最好在其他进程或线程中去做。而Node.js单线程的特点决定了无法很好的利用多核CPU，所以最好是让Nginx去做Gzip压缩。

### 13. Middleware Sequence

调整中间件顺序可用于简化某些业务请求的处理流程，减少某些请求的响应体大小。但调整时一定要明白Express与Koa中间件的执行顺序，不懂原理的话结果可能不会符合预期。

### 14. GC

新生代频繁回收问题，原理参考一下JVM的垃圾回收

### 15. Safety

一定要做安防，不然网站入口挂了那就惨了。最好买一个web防火墙(百度云加速)

除了需要防御常规的跨站请求攻击，XSS，有时也需要考虑限流方案。

### 16. PM2

标配控件，可以考虑用一下pm2的日志插件 [pm2-logrotate](https://github.com/keymetrics/pm2-logrotate) 

但有一点需要注意，启动多个Node实例可能会产生同一事件被消费多次的问题，常见的特殊场景比如定时任务, fswatch

### 17. Template Engine

让代码更加优雅，方便单独调试静态页面，不必依赖于后端渲染。

### 18. Module Development

模块化开发，公共模块以及各种业务模块，但熟知的几个框架只有Thinkjs支持了多模块

### 19. Context Validator

最好将validator抽离出来，别混在业务逻辑里面，如Sails的input参数校验，koa不知道有没有类似的插件

参考: [ajv](https://ajv.js.org/)

### 20. Emit

观察者模式的实现，用于解耦程序，不过要注意`EventEmitter.emit`是同步方法呦。当然也可以借助MQ实现订阅发布。

参考一下Node.js中模块实现，看看哪些地方用到了Emit

### 21. Array + Promise

并发操控数组元素，实现类似Spark中并行处理的效果

### 22. Cluster Mode

[Node.js Cluster](http://nodejs.cn/api/cluster.html) 工作模式，总结一下哪些框架支持该种运行模式，常见的应用场景有图片渲染(不可能为每个请求都创建一个puppeteer/phantom实例)

Sails.js支不支持该模式？反正 [Egg](https://eggjs.org/en/advanced/cluster-client.html) 支持

其中也涉及到一个问题 -- 惊群

### 23. Memory Usage

要注意内存溢出或内存使用不当的问题

- Nested setTimeout
- Closure
- FileReader

## 必读物

- [你不知道的Node.js性能优化](https://zhuanlan.zhihu.com/p/50055740)
- [Node.js的10个性能优化技巧](https://www.jb51.net/article/52187.htm)
- [基于Node.js的苏宁电商核心业务实践](https://myslide.cn/slides/9796)
- [苏宁Nodejs性能优化实战](https://mp.weixin.qq.com/s/JxRO5BhJai-tT6xWvFpKgQ)
- [数组的遍历你都会用了，那Promise版本的呢](https://segmentfault.com/a/1190000014598785)
- [evantahler/background_jobs_node](https://github.com/evantahler/background_jobs_node/blob/master/3-local.js)
- [Compare with pm2 and Forever](http://strong-pm.io/compare/)
- [最快的PHP路由](http://www.symfonychina.com/blog/new-in-symfony-4-1-fastest-php-router)
- [中间件执行模块koa-compose源码分析](https://segmentfault.com/a/1190000013447551)
- [以中间件，路由，跨进程事件的姿势使用WebSocket](https://segmentfault.com/a/1190000016914790)
- [4类 JavaScript 内存泄漏及如何避免](https://jinlong.github.io/2016/05/01/4-Types-of-Memory-Leaks-in-JavaScript-and-How-to-Get-Rid-Of-Them/)
- [How JavaScript works: memory management + how to handle 4 common memory leaks](https://blog.sessionstack.com/how-javascript-works-memory-management-how-to-handle-4-common-memory-leaks-3f28b94cfbec)
- [JavaScript内存泄漏教程](http://www.ruanyifeng.com/blog/2017/04/memory-leak.html)
- [The beautiful thing called EventEmitter](https://dev.to/tunaxor/the-beautiful-thing-called-eventemitter-23ei)
- [How to validate user input in a Node.js application](https://rethinkdb.com/blog/validation-techniques/)
- [使用Node.js实现文件流转存服务](https://github.com/andycall/blog/issues/1)
- [树结构遍历 —— 深度优先和广度优先](https://www.pandashen.com/2018/07/02/20180702122923/)
- [Koa源码分析](https://www.pandashen.com/2018/09/02/20180902141819/)
- [爬虫的广度优先和深度优先算法](https://www.cnblogs.com/wangshuyi/p/6734523.html)
- [i0natan/nodebestpractices](https://github.com/i0natan/nodebestpractices)
- [Egg.js设计原则](https://eggjs.org/zh-cn/intro/index.html)
- [Backpressuring in Streams](https://nodejs.org/en/docs/guides/backpressuring-in-streams/)
- [Express中间件摆放顺序之坑](https://go.kieran.top/post/34/)