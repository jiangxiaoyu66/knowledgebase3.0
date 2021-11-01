

## class组件有什么缺点？

1、代码编写麻烦，如要写constructor，要额外绑定this，setState语法糖写法冗余。class学习成本高

2、复用有状态的组件太难。写法不利于达到高内聚低耦合的要求，好比生命周期，didmount订阅后在willunmount中还要卸载，代码不写在一块，想要抽离就比较麻烦。想要把整个组件看一遍，复用成本高





**Hook 的每次调用都有一个完全独立的 state**



## 自定义hook：

如果函数的名字以 use 开头，并且调用了其他的 Hook，则就称其为一个自定义 Hook

```js
function AutoFocusIpt() {
  const inputEl = useRef(null);
  const useEffect(() => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  }, []);
  return (
    <>
      <input ref={inputEl} type="text" />
    </>
  );
}
```





## 2. 为什么 useState 要使用数组而不是对象

useState 的用法：

```javascript
const [count, setCount] = useState(0)
```

可以看到 useState 返回的是一个数组，那么为什么是返回数组而不是返回对象呢？

这里用到了解构赋值，所以先来看一下ES6 的解构赋值：

##### 数组的解构赋值

```javascript
const foo = [1, 2, 3];
const [one, two, three] = foo;
console.log(one);	// 1
console.log(two);	// 2
console.log(three);	// 3
复制代码
```

##### 对象的解构赋值

```javascript
const user = {
  id: 888,
  name: "xiaoxin"
};
const { id, name } = user;
console.log(id);	// 888
console.log(name);	// "xiaoxin"
```

看完这两个例子，答案应该就出来了：

- 如果 useState 返回的是数组，那么使用者可以对数组中的元素命名，代码看起来也比较干净
- 如果 useState 返回的是对象，**在解构对象的时候必须要和 useState 内部实现返回的对象同名**，想要使用多次的话，必须得设置别名才能使用返回值

下面来看看如果 useState 返回对象的情况：

```javascript
// 第一次使用
const { state, setState } = useState(false);
// 第二次使用
const { state: counter, setState: setCounter } = useState(0) 
复制代码
```

这里可以看到，返回对象的使用方式还是挺麻烦的，更何况实际项目中会使用的更频繁。 **总结：\**useState 返回的是 array 而不是 object 的原因就是为了\**降低使用的复杂度**，返回数组的话可以直接根据顺序解构，而返回对象的话要想使用多次就需要定义别名了。



## 4. React Hook 的使用限制有哪些？

React Hooks 的限制主要有两条：

- 不要在循环、条件或嵌套函数中调用 Hook；
- 在 React 的函数组件中调用 Hook。

那为什么会有这样的限制呢？Hooks 的设计初衷是为了改进 React 组件的开发模式。在旧有的开发模式下遇到了三个问题。

- 组件之间难以复用状态逻辑。过去常见的解决方案是高阶组件、render props 及状态管理框架。
- 复杂的组件变得难以理解。生命周期函数与业务逻辑耦合太深，导致关联部分难以拆分。
- 人和机器都很容易混淆类。常见的有 this 的问题，但在 React 团队中还有类难以优化的问题，希望在编译优化层面做出一些改进。

这三个问题在一定程度上阻碍了 React 的后续发展，所以为了解决这三个问题，Hooks **基于函数组件**开始设计。然而第三个问题决定了 Hooks 只支持函数组件。

那为什么不要在循环、条件或嵌套函数中调用 Hook 呢？因为 **Hooks 的设计是基于数组实现。在调用时按顺序加入数组中，如果使用循环、条件或嵌套函数很有可能导致数组取值错位，执行错误的 Hook。**当然，实质上 React 的源码里不是数组，是链表。

这些限制会在编码上造成一定程度的心智负担，新手可能会写错，为了避免这样的情况，可以引入 ESLint 的 Hooks 检查插件进行预防。

### 5. useEffect 与 useLayoutEffect 的区别

**（1）共同点**

- **运用效果：** useEffect 与 useLayoutEffect 两者都是用于处理副作用，这些副作用包括改变 DOM、设置订阅、操作定时器等。在函数组件内部操作副作用是不被允许的，所以需要使用这两个函数去处理。
- **使用方式：** useEffect 与 useLayoutEffect 两者底层的函数签名是完全一致的，都是调用的 mountEffectImpl方法，在使用上也没什么差异，基本可以直接替换。

**（2）不同点**

- **使用场景：** useEffect 在 React 的渲染过程中是被异步调用的，用于绝大多数场景；而 useLayoutEffect 会在所有的 DOM 变更之后同步调用，**主要用于 操作DOM、调整样式**、避免页面闪烁等问题。也正因为是同步处理，所以需要避免在 useLayoutEffect 做计算量较大的耗时任务从而造成阻塞。
- **使用效果：** useEffect是按照顺序执行代码的，改变屏幕像素之后执行（先渲染，后改变DOM），当改变屏幕内容时可能会产生闪烁；useLayoutEffect是改变屏幕像素之前就执行了（会推迟页面显示的事件，先改变DOM后渲染），不会产生闪烁。**useLayoutEffect总是比useEffect先执行。**

在未来的趋势上，两个 API 是会长期共存的，暂时没有删减合并的计划，需要开发者根据场景去自行选择。React 团队的建议非常实用，如果实在分不清，先用 useEffect，一般问题不大；如果页面有异常，再直接替换为 useLayoutEffect 即可。

### **6. React Hooks在平时开发中需要注意的问题和原因**

#### react是怎么保证多个useState的相互独立的？

还是看上面给出的ExampleWithManyStates例子，我们调用了三次useState，每次我们传的参数只是一个值（如42，‘banana’），我们根本没有告诉react这些值对应的key是哪个，那react是怎么保证这三个useState找到它对应的state呢？

答案是，react是根据useState出现的顺序来定的。我们具体来看一下：

```
  //第一次渲染
  useState(42);  //将age初始化为42
  useState('banana');  //将fruit初始化为banana
  useState([{ text: 'Learn Hooks' }]); //...

  //第二次渲染
  useState(42);  //读取状态变量age的值（这时候传的参数42直接被忽略）
  useState('banana');  //读取状态变量fruit的值（这时候传的参数banana直接被忽略）
  useState([{ text: 'Learn Hooks' }]); //...
复制代码
```

假如我们改一下代码：

```
let showFruit = true;function ExampleWithManyStates() {  const [age, setAge] = useState(42);    if(showFruit) {    const [fruit, setFruit] = useState('banana');    showFruit = false;  }   const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);复制代码
```

这样一来，

```
  //第一次渲染  useState(42);  //将age初始化为42  useState('banana');  //将fruit初始化为banana  useState([{ text: 'Learn Hooks' }]); //...  //第二次渲染  useState(42);  //读取状态变量age的值（这时候传的参数42直接被忽略）  // useState('banana');    useState([{ text: 'Learn Hooks' }]); //读取到的却是状态变量fruit的值，导致报错复制代码
```

鉴于此，react规定我们必须把hooks写在函数的最外层，不能写在ifelse等条件语句当中，来确保hooks的执行顺序一致。



（1）**不要在循环，条件或嵌套函数中调用Hook，必须始终在 React函数的顶层使用Hook**

这是因为React需要利用调用顺序来正确更新相应的状态，以及调用相应的钩子函数。一旦在循环或条件分支语句中调用Hook，就容易导致调用顺序的不一致性，从而产生难以预料到的后果。

**（2）使用useState时候，使用push，pop，splice等直接更改数组对象的坑**

使用push直接更改数组无法获取到新值，应该采用析构方式，但是在class里面不会有这个问题。代码示例：

```javascript
function Indicatorfilter() {  let [num,setNums] = useState([0,1,2,3])  const test = () => {    // 这里坑是直接采用push去更新num    // setNums(num)是无法更新num的    // 必须使用num = [...num ,1]    num.push(1)    // num = [...num ,1]    setNums(num)  }return (    <div className='filter'>      <div onClick={test}>测试</div>        <div>          {num.map((item,index) => (              <div key={index}>{item}</div>          ))}      </div>    </div>  )}class Indicatorfilter extends React.Component<any,any>{  constructor(props:any){      super(props)      this.state = {          nums:[1,2,3]      }      this.test = this.test.bind(this)  }  test(){      // class采用同样的方式是没有问题的      this.state.nums.push(1)      this.setState({          nums: this.state.nums      })  }  render(){      let {nums} = this.state      return(          <div>              <div onClick={this.test}>测试</div>                  <div>                      {nums.map((item:any,index:number) => (                          <div key={index}>{item}</div>                      ))}                  </div>          </div>      )  }}复制代码
```

（3）**useState设置状态的时候，只有第一次生效，后期需要更新状态，必须通过useEffect**

TableDeail是一个公共组件，在调用它的父组件里面，我们通过set改变columns的值，以为传递给TableDeail 的 columns是最新的值，所以tabColumn每次也是最新的值，但是实际tabColumn是最开始的值，不会随着columns的更新而更新：

```javascript
const TableDeail = ({    columns,}:TableData) => {    const [tabColumn, setTabColumn] = useState(columns) }// 正确的做法是通过useEffect改变这个值const TableDeail = ({    columns,}:TableData) => {    const [tabColumn, setTabColumn] = useState(columns)     useEffect(() =>{setTabColumn(columns)},[columns])}复制代码
```

**（4）善用useCallback**

父组件传递给子组件事件句柄时，如果我们没有任何参数变动可能会选用useMemo。但是每一次父组件渲染子组件即使没变化也会跟着渲染一次。

**（5）不要滥用useContext**

可以使用基于 useContext 封装的状态管理工具。

### 7. React Hooks 和生命周期的关系？

**函数组件** 的本质是函数，没有 state 的概念的，因此**不存在生命周期**一说，仅仅是一个 **render 函数**而已。 但是引入 **Hooks** 之后就变得不同了，它能让组件在不使用 class 的情况下拥有 state，所以就有了生命周期的概念，所谓的生命周期其实就是 `useState`、 `useEffect()` 和 `useLayoutEffect()` 。

即：**Hooks 组件（使用了Hooks的函数组件）有生命周期，而函数组件（未使用Hooks的函数组件）是没有生命周期的**。

下面是具体的 class 与 Hooks 的**生命周期对应关系**：

- `constructor`：函数组件不需要构造函数，可以通过调用 `**useState 来初始化 state**`。如果计算的代价比较昂贵，也可以传一个函数给 `useState`。

```javascript
const [num, UpdateNum] = useState(0)复制代码
```

- `getDerivedStateFromProps`：一般情况下，我们不需要使用它，可以在**渲染过程中更新 state**，以达到实现 `getDerivedStateFromProps` 的目的。

```javascript
function ScrollView({row}) {  let [isScrollingDown, setIsScrollingDown] = useState(false);  let [prevRow, setPrevRow] = useState(null);  if (row !== prevRow) {    // Row 自上次渲染以来发生过改变。更新 isScrollingDown。    setIsScrollingDown(prevRow !== null && row > prevRow);    setPrevRow(row);  }  return `Scrolling down: ${isScrollingDown}`;}复制代码
```

React 会立即退出第一次渲染并用更新后的 state 重新运行组件以避免耗费太多性能。

- `shouldComponentUpdate`：可以用 `**React.memo**` 包裹一个组件来对它的 `props` 进行浅比较

```javascript
const Button = React.memo((props) => {  // 具体的组件});复制代码
```

注意：`**React.memo 等效于 **``**PureComponent**`，它只浅比较 props。这里也可以使用 `useMemo` 优化每一个节点。

- `render`：这是函数组件体本身。
- `componentDidMount`, `componentDidUpdate`： `useLayoutEffect` 与它们两的调用阶段是一样的。但是，我们推荐你**一开始先用 useEffect**，只有当它出问题的时候再尝试使用 `useLayoutEffect`。`useEffect` 可以表达所有这些的组合。

```javascript
// componentDidMountuseEffect(()=>{  // 需要在 componentDidMount 执行的内容}, [])useEffect(() => {   // 在 componentDidMount，以及 count 更改时 componentDidUpdate 执行的内容  document.title = `You clicked ${count} times`;   return () => {    // 需要在 count 更改时 componentDidUpdate（先于 document.title = ... 执行，遵守先清理后更新）    // 以及 componentWillUnmount 执行的内容         } // 当函数中 Cleanup 函数会按照在代码中定义的顺序先后执行，与函数本身的特性无关}, [count]); // 仅在 count 更改时更新复制代码
```

**请记得 React 会等待浏览器完成画面渲染之后才会延迟调用 ，因此会使得额外操作很方便**

- `componentWillUnmount`：相当于 `useEffect `里面返回的 `cleanup` 函数

```javascript
// componentDidMount/componentWillUnmountuseEffect(()=>{  // 需要在 componentDidMount 执行的内容  return function cleanup() {    // 需要在 componentWillUnmount 执行的内容        }}, [])复制代码
```

- `componentDidCatch` and `getDerivedStateFromError`：目前**还没有**这些方法的 Hook 等价写法，但很快会加上。

| **class 组件**           | **Hooks 组件**            |
| ------------------------ | ------------------------- |
| constructor              | useState                  |
| getDerivedStateFromProps | useState 里面 update 函数 |
| shouldComponentUpdate    | useMemo                   |
| render                   | 函数本身                  |
| componentDidMount        | useEffect                 |
| componentDidUpdate       | useEffect                 |
| componentWillUnmount     | useEffect 里面返回的函数  |
| componentDidCatch        | 无                        |
| getDerivedStateFromError | 无                        |





