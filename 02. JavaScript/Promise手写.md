## 手写Promise

promise本身分为几个模块：

1、状态管理，resolve和reject的回调队列，resolve和reject回调函数传入的参数

2、执行resolve回调队列的条件：	
		1、状态由pending改为resolved/rejected
		2、resolve要传的值不是promise，如果是promise，要等该promise执行完
		3、保证代码执行顺序在本轮循环的同步代码之后（setTimeout）

>a tall guy:
>为了保证在本轮循环事件中，resolve回调函数在同步代码之后执行，用的settimeout包起来了
>
>a tall guy:
>但是promise状态fullfilled之后，resolve的回调函数不应该 是微任务吗
>
>a tall guy:
>用settimeout包起来不就是宏任务了吗，如果保证resolve回调函数在本轮循环的微任务后执行呢
>
>



```js
const PENDING = "pending";
const RESOLVED = "resolved";
const REJECTED = "rejected";

function MyPromise(fn) {
  // 保存初始化状态
  var self = this;

  // 初始化状态
  this.state = PENDING;

  // 用于保存 resolve 或者 rejected 传入的值
  this.value = null;

  // 用于保存 resolve 的回调函数
  this.resolvedCallbacks = [];

  // 用于保存 reject 的回调函数
  this.rejectedCallbacks = [];

  // 状态转变为 resolved 方法
  function resolve(value) {
    // 判断传入元素是否为 Promise 值，如果是，则状态改变必须等待前一个状态改变后再进行改变
    if (value instanceof MyPromise) {
      return value.then(resolve, reject);
    }

    // 保证代码的执行顺序为本轮事件循环的末尾
    setTimeout(() => {
      // 只有状态为 pending 时才能转变，
      if (self.state === PENDING) {
        // 修改状态
        self.state = RESOLVED;

        // 设置传入的值
        self.value = value;

        // 执行回调函数
        self.resolvedCallbacks.forEach(callback => {
          callback(value);
        });
      }
    }, 0);
  }

  // 状态转变为 rejected 方法
  function reject(value) {
    // 保证代码的执行顺序为本轮事件循环的末尾
    setTimeout(() => {
      // 只有状态为 pending 时才能转变
      if (self.state === PENDING) {
        // 修改状态
        self.state = REJECTED;

        // 设置传入的值
        self.value = value;

        // 执行回调函数
        self.rejectedCallbacks.forEach(callback => {
          callback(value);
        });
      }
    }, 0);
  }

  // 将两个方法传入函数执行
  try {
    fn(resolve, reject);
  } catch (e) {
    // 遇到错误时，捕获错误，执行 reject 函数
    reject(e);
  }
}

MyPromise.prototype.then = function(onResolved, onRejected) {
  // 首先判断两个参数是否为函数类型，因为这两个参数是可选参数
  onResolved =
    typeof onResolved === "function"
      ? onResolved
      : function(value) {
          return value;
        };

  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : function(error) {
          throw error;
        };

  // 如果是等待状态，则将函数加入对应列表中
  if (this.state === PENDING) {
    this.resolvedCallbacks.push(onResolved);
    this.rejectedCallbacks.push(onRejected);
  }

  // 如果状态已经凝固，则直接执行对应状态的函数

  if (this.state === RESOLVED) {
    onResolved(this.value);
  }

  if (this.state === REJECTED) {
    onRejected(this.value);
  }
};


```



## 手写Promise.then

思路：

1、如果then的参数不是函数，转为函数

2、判断promise当前的状态，pending则把then的两个参数函数推入回调队列；fullfilled则执行参数中第一个函数，rejected则执行第二个





## 手写Promise.all

注意三点：

1. 很多人不知道需要返回一个**Promise**对象，
2. 最好判断一下传入的参数是否为数组
3. 有很多人会分别判断数组里面的promise和常量， 但是**里面的顺序会乱**

```js
function PromiseAll(promiseArray) {    //返回一个Promise对象
     return new Promise((resolve, reject) => {
     
        if (!Array.isArray(promiseArray)) {                        //传入的参数是否为数组
            return reject(new Error('传入的参数不是数组！'))
        }

        const res = []
        let counter = 0                         //设置一个计数器
        for (let i = 0; i < promiseArr.length; i++) {
            Promise.resolve(promiseArr[i]).then(value => {
                counter++                  //使用计数器返回 必须使用counter
                res[i] = value   // 保证顺序
                if (counter === promiseArr.length) {
                    resolve(res)
                }
            }).catch(e => reject(e))
        }
    })
}

```





## 手写Promise.race

  `Promise.race()`与`all`一样将多个`Promise`实例包装为一个新的`Promise`实例。

```js
Promise.race = function (args) {
  return new Promise((resolve, reject) => {
    for (let i = 0, len = args.length; i < len; i++) {
      args[i].then(resolve, reject)
    }
  })
}

```



