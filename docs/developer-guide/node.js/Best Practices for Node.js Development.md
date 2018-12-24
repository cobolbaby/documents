[TOC]

## Node.js Optimize

### 1. Node.js Version

新版的Node.js在性能上只会更优

### 2. Use built-in modules

尽量使用系统内置的模块，而不引入第三方模块

### 3. Static Resource

静态资源让Nginx去管，也方便以后迁移至CDN

### 4. Router Cache

针对静态路由规则，可以采用Map优化，选择框架的时候，一定要注意其是否针对路由匹配有做过相关优化

针对Restful动态路由，将逐条匹配优化为单条匹配，效率应该能提升两个数量级

让单线程的Node.js做大量的正则匹配是很占用时间的

### 5. JWT -- Avoid Session

不需要每来一个请求都要请求一次Redis，这一点在PHP 中更明显。无法常驻内存的程序最大的问题就是没有办法做到连接的复用。

### 6. Sync I/O

用异步尽量别用同步代码，毕竟Node.js是单线程的，同步代码会阻塞线程执行。同时也可能造成内存溢出

- 同步文件读取
- 正则匹配
- 10w+循环赋值
- Gzip压缩

### 7. Promise Performance

推荐使用`Bluebird`实现的`Promise`

其中抛出的异常一定要catch，不然可能会造成程序异常退出

### 8. Parallel Process

能并行绝对不串行，缩短响应时间，如果是在做爬虫，请注意别太嚣张，不然很容易触发对方的底线

### 9. Stream

文件转存，请求代理，建议用Stream

### 10. JSON Serialize

JSON序列化优化在Java中很常见，在其他语言中其实也是个话题，类似的还有Protobuffer协议性能问题

### 11. Template Cache

针对模板，先考虑用缓存，然后再考虑重新组装。尤其是做后端渲染的时候，一定要利用好。

### 12. Gzip Compress

适当压缩会减少请求的网络时延，在弱网络环境下必要做,，但最好是让Nginx去做Gzip压缩。

### 13. Middleware Sequence

考虑调整中间件顺序，请求能早响应就别拖到最后。

### 14. GC

新生代频繁回收问题

### 15. Safety

做好跨站脚本攻击的防护，一定要做异常防护，不然网站入口就挂了。不过最好也买一个web防火墙(百度云加速)

### 16. PM2

标配控件，可以考虑用一下pm2的日志插件 [pm2-logrotate](https://github.com/keymetrics/pm2-logrotate) 

但有一点需要注意，启动多个Node实例可能会产生同一事件被消费多次的问题，常见的特殊场景比如定时任务, fswatch

### 17. Template Engine

让代码更加优雅

### 18. Module Development

模块化开发，公共模块以及各种业务模块，但熟知的几个框架只有Thinkjs支持了多模块

### 19. Context Validator

最好将validator抽象出来，别混在业务逻辑里面，如sails的input参数校验，koa不知道有没有类似的插件

### 20. Emit

观察者模式的实现，用于解耦程序，不过要注意`EventEmitter.emit`是同步方法呦。当然也可以借助MQ实现订阅发布。

参考一下Node.js中模块实现，看看哪些地方用到了Emit

### 21. Array + Promise

并发操控数组元素，实现类似Spark中并行处理的效果

### 22. Cluster Mode

[Node.js Cluster](http://nodejs.cn/api/cluster.html) 工作模式

其中也涉及到一个问题 -- 惊群

## Reference

- [Node.js内存溢出-process out of memory 问题的处理](https://niefengjun.cn/blog/62277f34c6ed048fac91583bd6214ddb.html)
- [你不知道的Node.js性能优化](https://zhuanlan.zhihu.com/p/50055740)
- [Node.js的10个性能优化技巧](https://www.jb51.net/article/52187.htm)
- [基于Node.js的苏宁电商核心业务实践](https://myslide.cn/slides/9796)
- [苏宁Nodejs性能优化实战](https://mp.weixin.qq.com/s/JxRO5BhJai-tT6xWvFpKgQ)
- [数组的遍历你都会用了，那Promise版本的呢](https://segmentfault.com/a/1190000014598785)
- [background_jobs_node](https://github.com/evantahler/background_jobs_node/blob/master/3-local.js)
- [Compare with pm2 and Forever](http://strong-pm.io/compare/)
- [最快的PHP路由](http://www.symfonychina.com/blog/new-in-symfony-4-1-fastest-php-router)
- [中间件执行模块koa-Compose源码分析](https://segmentfault.com/a/1190000013447551)