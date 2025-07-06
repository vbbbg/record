### 背景
公司的单页应用实现了一个多页签功能，这个功能就像浏览器一样，让项目可以打开多个路由页面  

依赖的第三方库是 `react-router-cache-route`

发布后，发现某些情况下，调用 [dropByCacheKey](https://github.com/CJY0208/react-router-cache-route?tab=readme-ov-file#drop-cache) 后，这个页面的缓存还存在内容中  

这导致用户在需要连续提交新建用户表单的场景时，这个页面的表单数据不会重置  

公司的同事几经排查，无法解决  

后来，这个 bug 转到我手上了  

### 暂时的解决
我不懂 `react-router`, 我理解不了 `react-router-cache-route` 的实现机制  

我使用 npm link 在源码上盲目调试了半天，只知道 cache 不正确，貌似被新的 cache 覆盖了  

路由应该只生成一次才对，不存在被覆盖的可能

无奈，只能通过一些技巧，巧妙的绕过这个bug解决了问题  

具体原因无人知晓，我心里有颗石头  

### 理解 Cache
几经思考，觉得还得从原理入手，于是我便找到了 `react-router@4.3.1` 的源码  

原来这个版本的源码就几行代码，主要逻辑就是： Switch 找到路由数组中第一个匹配的路由, Route 负责渲染  

为什么我要说第一个呢？这就要说到 `react-router-cache-route` 的实现逻辑了

它的实现原理就是，CacheSwitch 找到第一个路由后，并不会停止，还会继续循环，会将路由传给 CacheRoute  

CacheRoute 会在 constructor 时，创建 cacheKey，并将实例存储起来

routes 大概就是下面这一坨：

```javascript
const routes = [
  {
    path: "/sandwiches",
    component: Sandwiches
  },
  {
    path: "/tacos/:id",
    component: Tacos,
  }
];
```



### 根治
结合如前所说
1. cache 被覆盖了
2. CacheSwitch 会一直循环，CacheRoute 会创建 cacheKey  

我将 console 打在 cacheKey 的生成上，过滤有问题的路由，我发现有问题的路由 cacheKey 打印了两次

很明显 CacheSwitch 在循环的过程中，生成了被新生成的 key 覆盖了

于是我改了 [cacheKey](https://github.com/CJY0208/react-router-cache-route?tab=readme-ov-file#cacheroute-props) 的实现逻辑，问题得以解决