## localStorage与sessionStorage

`localStorage`和`sessionStorage`是`HTML5`提供的对于`Web`本地存储的解决方案。



### 相同点

1. 都是html5提供的针对Web本地存储的方案
2. 都不会和cookie一样，在浏览器请求时自动携带
3. 都是键值对和String的形式存储
4. 容量有限，都在5M左右，具体大小随浏览器不同而不同



### 不同点

localStorage属于持久化的本地存储，不手动删除则数据永不过期；sessionStorage则在浏览器标签页关闭后就失效

