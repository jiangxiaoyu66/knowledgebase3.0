##### Hybrid如何通信的?

> 1. API注入，原理其实就是 Native 获取 JavaScript环境上下文，并直接在上面挂载对象或者方法，使 js 可以直接调用，Android 与 IOS 分别拥有对应的挂载方式
> 2. WebView 中的 prompt/console/alert 拦截，通常使用 prompt，因为这个方法在前端中使用频率低，比较不会出现冲突
> 3. WebView URL Scheme 跳转拦截

![img](图片/e3d7ed6db7524dca91269a9464fba968tplv-k3u1fbpfcp-watermark.awebp)



作者：程洋cYang
链接：https://juejin.cn/post/6896810576778166280
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。