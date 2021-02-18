## 浅析http缓存机制
http缓存机制是web性能优化的重要手段之一，对于从事前端开发的同学来说，http缓存机制是必备的基础知识技能。

1. 缓存规则解析
  为了方便理解，我们假设浏览器存在一个缓存数据库，用于存储数据。在客户端第一次发送请求时，此时，缓存数据库中没有任何对应的缓存，需要请求服务器。服务器返回相应的缓存标识和数据后，将其存储至浏览器的缓存数据库中。
  
  ![客户端第一次请求数据](https://github.com/yzbest111/Network/blob/dev/images/http1.png)
  
  http缓存可分为强缓存和协商缓存，下面通过时序图，让我们对其有一个直观的感受
  
  以下为强缓存的时序图
  ![强缓存，缓存命中](https://github.com/yzbest111/Network/blob/dev/images/http强缓存命中.png)
  ![强缓存，缓存未命中](https://github.com/yzbest111/Network/blob/dev/images/http强缓存未命中.png)
  
  以下为协商缓存的时序图
  ![协商缓存，缓存命中](https://github.com/yzbest111/Network/blob/dev/images/协商缓存命中.png)
  ![协商缓存，缓存未命中](https://github.com/yzbest111/Network/blob/dev/images/协商缓存未命中.png)
