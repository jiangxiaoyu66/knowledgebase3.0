参考：[Fiber](https://juejin.cn/post/6943896410987659277#heading-12)

## 前置知识



我们知道，在浏览器中，页面是一帧一帧绘制出来的，渲染的帧率与设备的刷新率保持一致。一般情况下，设备的屏幕刷新率为1s 60次，当每秒内绘制的帧数（FPS）超过60时，页面渲染是流畅的；而当FPS小于60时，会出现一定程度的卡顿现象。



### 16版本以前

```
16版本前存在的问题：
这种遍历是递归调用，所以不能中断，中断后也不能恢复了。一旦递归的层级太深，递归会一直占用主线程，其他事件就得不到执行，也就会出现卡顿的现象。
```



在 react16 引入 Fiber 架构之前，react 会采用递归对比虚拟DOM树，找出需要变动的节点，然后同步更新它们，这个过程 react 称为`reconcilation`（协调）。在`reconcilation`期间，react 会一直占用浏览器资源，会导致用户触发的事件得不到响应。实现的原理如下所示：

![undefined](图片/994d70ab12f14d079e6c0b98d0e1b326tplv-k3u1fbpfcp-watermark.awebp)

这里有7个节点，B1、B2 是 A1 的子节点，C1、C2 是 B1 的子节点，C3、C4 是 B2 的子节点。传统的做法就是采用深度优先遍历去遍历节点，具体代码如下：

```js
const root = {
  key: 'A1',
  children: [{
    key: 'B1',
    children: [{
      key: 'C1',
      children: []
    }, {
      key: 'C2',
      children: []
    }]
  }, {
    key: 'B2',
    children: [{
      key: 'C3',
      children: []
    }, {
      key: 'C4',
      children: []
    }]
  }]
}
const walk = dom => {
  console.log(dom.key)
  dom.children.forEach(child => walk(child))
}
walk(root)

复制代码
```

打印：

```
A1
B1
C1
C2
B2
C3
C4
```

这种遍历是递归调用，执行栈会越来越深，而且不能中断，中断后就不能恢复了。递归如果非常深，就会十分卡顿。如果递归花了100ms，则这100ms浏览器是无法响应的，代码执行时间越长卡顿越明显。传统的方法存在不能中断和执行栈太深的问题。











## fiber

### 什么是fibers

fiber，我们把它叫作协程，它是一种比线程更小的单位。每个线程中有多个协程，协程可以随时中断和继续。

从实现上看，他其实一种链式的数据结构。

Fiber 可以说是一个执行单元，也可以说是一种数据结构

#### 一个执行单元

Fiber 可以理解为一个执行单元，每次执行完一个执行单元，react 就会检查现在还剩多少时间，如果没有时间则将控制权让出去。



#### 一种数据结构

Fiber 还可以理解为是一种数据结构，React Fiber 就是采用链表实现的。每个 Virtual DOM 都可以表示为一个 fiber，如下图所示，每个节点都是一个 fiber。一个 fiber包括了 child（第一个子节点）、sibling（兄弟节点）、return（父节点）等属性

![undefined](图片/74fe5e7d1dc449568448b462abcbff6etplv-k3u1fbpfcp-watermark.awebp)



### requestAnimationFrame

在 Fiber 中使用到了[requestAnimationFrame](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWindow%2FrequestAnimationFrame)，它是浏览器提供的绘制动画的 api 。它要求浏览器在下次重绘之前（即下一帧）调用指定的回调函数更新动画。

### requestIdleCallback

[requestIdleCallback](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWindow%2FrequestIdleCallback) 也是 react Fiber 实现的基础 api 。我们希望能够快速响应用户，让用户觉得够快，不能阻塞用户的交互，`requestIdleCallback`能使开发者在主事件循环上执行后台和低优先级的工作，而不影响延迟关键事件，如动画和输入响应。正常帧任务完成后没超过16ms，说明有多余的空闲时间，此时就会执行`requestIdleCallback`里注册的任务。

```js
window.requestIdleCallback(callback, { timeout: 1000 })
```







## Fiber执行原理

从根节点开始渲染和调度的过程可以分为两个阶段：render 阶段、commit 阶段。

- render 阶段：这个阶段是可中断的，会找出所有节点的变更
- commit 阶段：这个阶段是不可中断的，会执行所有的变更

### render阶段

此阶段会找出所有节点的变更，如节点新增、删除、属性变更等，这些变更 react 统称为副作用（effect），此阶段会构建一棵`Fiber tree`，以虚拟dom节点为维度对任务进行拆分，即一个虚拟dom节点对应一个任务，最后产出的结果是`effect list`，从中可以知道哪些节点更新、哪些节点增加、哪些节点删除了。

#### 遍历流程

`React Fiber`首先是将虚拟DOM树转化为`Fiber tree`，因此每个节点都有`child`、`sibling`、`return`属性，遍历`Fiber tree`时采用的是后序遍历方法：

先子级，后兄弟，然后叔叔。完成所有节点就表示遍历完成。

![undefined](图片/3e617a3507074e498318332b579cd634tplv-k3u1fbpfcp-watermark.awebp)












## reconciliation阶段

reconciliation阶段包含的主要工作是对current tree 和 new tree 做diff计算，找出变化部分。进行遍历、对比等是可以中断，歇一会儿接着再来。

commit阶段是对上一阶段获取到的变化部分应用到真实的DOM树中，是一系列的DOM操作。不仅要维护更复杂的DOM状态，而且中断后再继续，会对用户体验造成影响。在普遍的应用场景下，此阶段的耗时比diff计算等耗时相对短。





















