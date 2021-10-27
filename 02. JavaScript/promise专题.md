# Promise

 `Promise`的特点：
	 promise有三个状态：`pending`、`fulfilled`和`rejected`，三种状态只受内部逻辑代码控制，不受外界影响。状态只能由pending变成fullfilled或rejected



 `Promise`的缺点：

（1）无法取消`Promise`，一旦新建就会立即执行；

（2）如果不设置回调函数，`Promise`内部抛出的错误不会反应到外部(除非用catch抛错进行链式传递，才能获取错误信息)；

（3）处于`pending`状态时，无法得知目前进展（刚开始还是快结束）。



## 基本用法

## Promise.prototype.then()

  `then`方法**返回的是一个新的`Promise`实例，**可以采用链式写法，在其后面继续调用`then`。

​		第二个参数`then(null, rejection)`或者`then(undefined, rejection)`，可以设置发生错误时候的回调

------

## Promise.prototype.catch()

  `Promise.prototype.catch()`方法用于发生错误时的回调函数，与`then(null, rejection)`或者`then(undefined, rejection)`相同。

  `catch`方法返回的也还是一个`Promise`对象，且`catch`方法也可能产生错误，除非后面再接一个catch，否则无法将错误传递到外部。

```js
getJSON('/posts.json')
.then((posts) => {
  //  ...some code
})
.catch((error) => {
  //  处理getJSON和前一个回调函数then运行时发生的错误
})
```

**注意**：最好不要用`then`方法的第二个参数里面定义Reject状态的回调函数，而是使用`catch`来处理，这样的话`then`方法如果出错也将会被catch所捕获，写法上也更接近同步写法`try/catch`。

------

## Promise.prototype.finally()

1、`finally()` 方法返回一个Promise。在promise结束时，无论结果是fulfilled或者是rejected，都会执行指定的回调函数。这为在Promise是否成功完成后都需要执行的代码提供了一种方式，避免了同样的语句需要在then()和catch()中各写一次的情况。

2、`finally`方法的回调函数不接受任何参数，因此无法知道前面`Promise`的状态如何。

>如果你想在 promise 执行完毕后无论其结果是失败还是成功都做一些处理或清理时，finally() 方法可能是有用的。
>finally() 虽然与 .then(onFinally, onFinally) 类似，但它们不同的是：
>
>- 调用内联函数时，不需要多次声明该函数或为该函数创建一个变量保存它。
>- 由于无法知道promise的最终状态，所以finally的回调函数中不接收任何参数，它仅用于无论最终结果如何都要执行的情况。
>- 与`Promise.resolve(2).then(() => {}, () => {})` （resolved的结果为`undefined`）不同，`Promise.resolve(2).finally(() => {}) resolved`的结果为 2。
>- 同样，`Promise.reject(3).then(() => {}, () => {})` (resolved 的结果为`undefined`), `Promise.reject(3).finally(() => {}) rejected` 的结果为 3。

------

## **Promise.all()**



>`promise.all` 的特点：
>
>- 入参是个由`Promise`实例组成的数组
>- 返回值是个`promise`，因为可以使用`.then`
>- s如果全部成功，状态变为`resolved`, 并且返回值组成一个数组传给回调
>- 但凡有一个失败，状态变为`rejected`, 并将`error`返回给回调
>
>





例如：

```js
const thenable = {
    then: (resolve, reject) => {
        resolve(42);
    }
};

const p1 = Promise.resolve(thenable);
p1.then((value) => {
    console.log(value); //  42
});
```

上面代码中，`thenasble`对象的`then`方法执行后，对象`p1`的状态就变为`resolved`，从而立即执行最后那个`then`方法指定的回调函数，输出42。

  `Promise.all()`用于将多个`Promise`实例包装成一个新的`Promise`实例。

1、它接受一个数组或者有`Iterator`接口的对象作为参数。

```js
const p = Promise.all([p1,p2,p3]);
```

  假如其中有不是`Promise`实例的部分，会调用一个`Promise.resolve`方法转换为`Promise`实例。

2、三者状态都变成`fulfilled`，**P**才会变成`fulfilled`，此时**P1，P2，P3**的返回值组成一个数组，传递给**P**的回调函数；

3、只要其中有一个是`rejected`，**P**就变成`rejected`，而第一个变成`rejected`的实例的返回值，将会传递给**P**的回调函数。

**注意**：如果作为参数的`Promise`实例定义了自己的`catch`方法，那么它一旦`rejected`，将不会触发`Promise.all()`的`catch`方法。



栗子：

```
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result)
.catch(e => e);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.then(result => result)
.catch(e => e);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// 并不会返回["hello", Error: 报错了]
```

  上面代码，**P1**会resolve，**P2**首先变成`rejected`，但因为它有自己的`catch`方法，而`catch`方法返回的是一个新的`Promise`实例，因此**P2**实际指向的是这个`catch`方法返回的实例。这个实例在执行完`catch`后也会变成`resolved`，导致`Promise.all()`方法参数里的两个实例都会`resolved`，因此会调用`then`方法，而不是`catch`方法。

  如果**P2**没有自己的`catch`方法，才会调用`Promise.all()`的`catch`方法（如下所示）。

```
const p1 = new Promise((resolve, reject) => {
  resolve('hello');
})
.then(result => result);

const p2 = new Promise((resolve, reject) => {
  throw new Error('报错了');
})
.then(result => result);

Promise.all([p1, p2])
.then(result => console.log(result))
.catch(e => console.log(e));
// Error: 报错了
```

------

## Promise.race()

  `Promise.race()`与`all`一样将多个`Promise`实例包装为一个新的`Promise`实例。

```
const p = Promise.race([p1, p2, p3]);
```

  与`all`不同的是，只要参数中的实例有一个率先改变状态，`p`的状态就跟着改变，而那个实例的返回值就传递给`p`的回调函数。

>Promise.race([p1, p2, p3])里面哪个结果获得的快，就返回那个结果，不管结果本身是成功状态还是失败状态。
>
>例子：
>
>```js
>let p1 = new Promise((resolve, reject) => {
>  setTimeout(() => {
>    resolve('success')
>  },1000)
>})
>
>let p2 = new Promise((resolve, reject) => {
>  setTimeout(() => {
>    reject('failed')
>  }, 500)
>})
>
>Promise.race([p1, p2]).then((result) => {
>  console.log(result)
>}).catch((error) => {
>  console.log(error)  // 打开的是 'failed'
>})
>```
>
>
>
>

------

## Promise.allSettled()

  `Promise.allSettled()`方法接受一组`Promise`实例作为参数，包装为一个新`Promise`实例。

​		等到所有的参数实例都返回结果，无论结果状态如何，包装实例都会结束。

​		该方法返回的新的`Promise`实例，一旦结束，状态总是`fulfilled`。状态改变后，`Promise`监听函数接收到的参数是一个数组，每个成员对应一个传入`Promise.allSettled()`的`Promise`实例。

```js
const resolved = Promise.resolve(42);
const rejected = Promise.reject(-1);

const allSettledPromise = Promise.allSettled([resolved, rejected]);

allSettledPromise.then((results) => {
  console.log(results);
});
// [
//    { status: 'fulfilled', value: 42 },
//    { status: 'rejected', reason: -1 }
// ]
```

  上面代码中，`results`数组的每个成员都是一个对象，每个对象都有`status`属性，值只可能是`fulfilled`或者`rejected`。当是`fulfilled`时，对象有`value`属性；`rejected`时有`reason`属性。 可以使用简单的`Array.prorotype.filter()`筛选出成功或失败的请求，输出其结果或原因。

------

## Promise.any()

  **只要参数实例有一个变成`fulfilled`状态，包装的实例就会变成`fulfilled`；如果所有参数实例都变成`rejected`状态，包装实例才会变成`rejected`状态。**

```js
const resolved = Promise.resolve(42);
const rejected = Promise.reject(-1);
const alsoRejected = Promise.reject(Infinity);

Promise.any([resolved, rejected, alsoRejected]).then(function (result) {
  console.log(result); // 42
});

Promise.any([rejected, alsoRejected]).catch(function (results) {
  console.log(results); // [-1, Infinity]
});
```

------

## **Promise.resolve()**

  如果需要将现有的对象转为`Promise`对象，`Promise.resolve()`方法就起这个作用。

  `Promise.resolve()`等价于下面写法：

```js
Promise.resolve('foo');

new Promise(resolve => resolve('foo'));
```

  `Promise.resolve`方法的参数分成四种情况。

- 参数是一个`Promise`实例：不做任何事情，原封不动返回它。

- 参数是一个有then方法的对象：`Promise.resolve`方法会将这个对象转为`Promise`对象，然后立即执行`thenable`对象的`then`方法。

  >例如：
  >
  >```js
  >const thenable = {
  >    then: (resolve, reject) => {
  >        resolve(42);
  >    }
  >};
  >
  >const p1 = Promise.resolve(thenable);
  >p1.then((value) => {
  >    console.log(value); //  42
  >});
  >```
  >
  >上面代码中，`thenasble`对象的`then`方法执行后，对象`p1`的状态就变为`resolved`，从而立即执行最后那个`then`方法指定的回调函数，输出42。
  >
  >
  >
  >例子2：
  >
  >```js
  >const p1 = new Promise(function (resolve, reject) {
  >  // ...
  >});
  >
  >const p2 = new Promise(function (resolve, reject) {
  >  // ...
  >  resolve(p1);
  >})
  >```
  >
  >
  >上面代码中，p1和p2都是 Promise 的实例，但是p2的resolve方法将p1作为参数，即一个异步操作的结果是返回另一个异步操作。
  >
  >注意，这时p1的状态就会传递给p2，也就是说，p1的状态决定了p2的状态。如果p1的状态是pending，那么p2的回调函数就会等待p1的状态改变；如果p1的状态已经是resolved或者rejected，那么p2的回调函数将会立刻执行。
  >

  

- 参数不是具有`then`方法的对象，或根本不是个对象：直接把该参数传给then的第一个回调函数

如果参数是一个原始值，或一个没有`then`方法的对象，那么`Promise.resolve`方法返回一个新的`Promise`对象，状态为`resolved`。

```
const p = Promise.resolve('Hello');

p.then(function (s){
  console.log(s)
});
// Hello
```

- 不带有任何参数

这种情况下，会直接返回一个`resolved`状态`Promise`对象，所以想获得一个`Promise`对象，方便的方法是直接调用`Promise.resolve()`方法。需要注意的是，**`resolve()`的`Promise`对象，是在本轮“事件循环”（event-loop）结束时执行，而不是下一轮“事件循环”的开始时。**

```
setTimeout(function () {
  console.log('three');
}, 0);

Promise.resolve().then(function () {
  console.log('two');
});

console.log('one');

// one
// two
// three
```



>关于“**立即 resolved 的 Promise 是在本轮事件循环的末尾执行，总是晚于本轮事件循环的同步任务。**”这段话的理解：
>
>```js
>new Promise((resolve, reject) => {
>  resolve(1);
>  console.log(2);
>}).then(r => {
>  console.log(r);
>});
>```
>
>上面代码中，调用resolve(1)以后，后面的console.log(2)还是会执行，并且会首先打印出来。这是因为立即 resolved 的 Promise 是在本轮事件循环的末尾执行，总是晚于本轮循环的同步任务。
>
>




------

## Promise.reject()

  `Promise.reject(reason)`方法也会返回一个新的`Promise`实例，该实例的状态为`rejected`。需要注意的是`Promise.reject()`方法的参数，会原封不动作为`reject`的理由变成后续方法的参数。

```js
const p = Promise.reject('出错了');
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
const thenable = {
  then(resolve, reject) {
    reject('出错了');
  }
};

Promise.reject(thenable)
.catch(e => {
  console.log(e === thenable)
})
// true
```

上面代码中，`Promise.reject`方法的参数是一个`thenable`对象，执行以后，后面`catch`方法的参数不是`reject`抛出的“出错了”这个字符串，而是`thenable`对象。







## 