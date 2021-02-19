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
  
2. 强缓存
  > 从上述强缓存的时序图可知，强缓存，在缓存数据未失效的情况下，可以直接使用缓存数据，那么浏览器是如何判断缓存数据是否失效的呢？
  > 我们知道，浏览器在没有缓存数据的情况下，会向服务器请求数据，服务器便会将缓存规则和数据一同返回，**缓存规则信息会被包含在response header中**。
  > 对于强缓存来说，reponse header中会有两个字段来标明失效规则：Expires/Cache-Control。
  
  * Expires：Expires的值为服务器端返回的到期时间。即下一次请求时，请求时间小于服务器端返回的到期时间，则直接使用缓存数据。不过，Expires是http1.0的产物，现在的浏览器默认使用http1.1，所以它的作用基本忽略。另一个原因是到期时间是服务器端生成返回的，但是客户端时间可能跟服务端时间有误差，这就会导致缓存命中的误差。所以，http1.1采用Cache-Control来代替。

  * Cache-Control：Cache-Control的常见取值有**private、public、max-age、no-cache、no-store**，默认为private。
    > private：客户端可以缓存
    > public：客户端和代理服务器都可缓存（前端同学可认为public和private是一样的）
    > max-age=xxx：缓存的内容将在xxx秒后失效
    > no-cache：需要走协商缓存来验证缓存数据
    > no-store：所有内容都不会缓存，强缓存和协商缓存都不会触发
    > 
3. 协商缓存
  > 对比强缓存的时序图，我们可以发现，不论协商缓存是否失效，浏览器都需要请求服务器，即将缓存标识发送给服务器进行验证，如果未失效，则服务器只返回响应头，和304状态码，告知浏览器可使用缓存。
  > 如果经服务器端验证后失效，则重新返回响应头和响应主体。
  > 协商缓存的缓存标识如何传递是需要我们着重理解的，它一共分为两种标识：Last-Modified/if-Modified-Since和Etag/if-None-Match
  > 
  * Last-Modified/if-Modified-Since：
    > Last-Modified：服务器在响应请求时，告知浏览器资源的最后修改时间
    > if-Mo3dified-Since：浏览器再次请求资源时，会带上此字段告知服务器上次请求时该资源的最后修改时间。服务器收到请求后，发现有请求头的if-Modified-Since字段，会与请求资源的最后修改时间进行比对。若资源的最后修改时间大于if-Modified-Since，说明该资源已被修改过，需要返回最新内容，返回状态码200；若资源的最后修改时间小于等于if-Modified-Since的值，说明资源没有修改，返回304状态码，告知浏览器可以使用所缓存的数据
    
  * Etag/if-None-Match：优先级高于Last-Modified/if-Modified-Since
    > Etag：服务器响应请求时，告知浏览器当前资源在服务器端端唯一标识（生成规则由服务器端决定）
    > if-None-Match：再次请求服务器时，通过此字段告知服务器端缓存数据的唯一标识。服务器收到请求后，发现请求头有if-None-Match字段，则会取其值与被请求资源的唯一标识进行比对。如果不同，说明资源又被改动过，则会返回最新的请求资源和200状态码。如果相同，说明资源没有修改，返回304状态码，告知浏览器可以使用缓存的数据。

#### 总结
对于强缓存，服务器告知浏览器一个缓存时间，在缓存时间内，下次请求，直接用缓存，不在时间内，执行协商缓存策略。 对于协商缓存，将缓存信息中的Etag和Last-Modified通过请求发送给服务器，由服务器校验，返回304状态码时，浏览器直接使用缓存。
