### 1. React-Router的实现原理是什么？

客户端路由实现的思想：

- 基于 hash 的路由：通过监听`hashchange`事件，感知 hash 的变化
  - 改变 hash 可以直接通过 location.hash=xxx
- 基于 H5 history 路由：
  - 改变 url 可以通过 history.pushState 和 resplaceState 等，会将URL压入堆栈，同时能够应用 `history.go()` 等 API
  - 监听 url 的变化可以通过自定义事件触发实现

**react-router 实现的思想：**

- 基于 `history` 库来实现上述不同的客户端路由实现思想，并且能够保存历史记录等，磨平浏览器差异，上层无感知
- 通过维护的列表，在每次 URL 发生变化的回收，通过配置的 路由路径，匹配到对应的 Component，并且 render

### 2. 如何配置 React-Router 实现路由切换

**（1）使用`<Route>` 组件**

路由匹配是通过比较 `<Route>` 的 path 属性和当前地址的 pathname 来实现的。当一个 `<Route>` 匹配成功时，它将渲染其内容，当它不匹配时就会渲染 null。没有路径的 `<Route>` 将始终被匹配。

```javascript
// when location = { pathname: '/about' }
<Route path='/about' component={About}/> // renders <About/>
<Route path='/contact' component={Contact}/> // renders null
<Route component={Always}/> // renders <Always/>
```

**（2）结合使用 `<Switch>` 组件和 `<Route>` 组件**

`<Switch>` 用于将 `<Route>` 分组。

```javascript
<Switch>
    <Route exact path="/" component={Home} />
    <Route path="/about" component={About} />
    <Route path="/contact" component={Contact} />
</Switch>
```

`<Switch>` 不是分组 `<Route>` 所必须的，但他通常很有用。 一个 `<Switch>` 会遍历其所有的子 `<Route>`元素，并仅渲染与当前地址匹配的第一个元素。

**（3）使用 `<Link>、 <NavLink>、<Redirect>` 组件**

`<Link>` 组件来在你的应用程序中创建链接。无论你在何处渲染一个`<Link>` ，都会在应用程序的 HTML 中渲染锚（`<a>`）。

```javascript
<Link to="/">Home</Link>   
// <a href='/'>Home</a>
复制代码
```

 是一种特殊类型的  当它的 to属性与当前地址匹配时，可以将其定义为"活跃的"。

```javascript
// location = { pathname: '/react' }
<NavLink to="/react" activeClassName="hurray">
    React
</NavLink>
// <a href='/react' className='hurray'>React</a>
复制代码
```

当我们想强制导航时，可以渲染一个`<Redirect>`，当一个`<Redirect>`渲染时，它将使用它的to属性进行定向。

### 3. React-Router怎么设置重定向？

使用`<Redirect>`组件实现路由的重定向：

```javascript
<Switch>
  <Redirect from='/users/:id' to='/users/profile/:id'/>
  <Route path='/users/profile/:id' component={Profile}/>
</Switch>
复制代码
```

当请求 `/users/:id` 被重定向去 `'/users/profile/:id'`：

- 属性 `from: string`：需要匹配的将要被重定向路径。
- 属性 `to: string`：重定向的 URL 字符串
- 属性 `to: object`：重定向的 location 对象
- 属性 `push: bool`：若为真，重定向操作将会把新地址加入到访问历史记录里面，并且无法回退到前面的页面。

### 4. react-router 里的 Link 标签和 a 标签的区别

从最终渲染的 DOM 来看，这两者都是链接，都是 标签，区别是∶

 `<Link>`只会刷新`<Route>`对应的页面，`<a>`会刷新整个页面

> `<Link>`是react-router 里实现路由跳转的链接，一般配合`<Route>` 使用，react-router接管了其默认的链接跳转行为，相比于传统的页面跳转，`<Link>` 的“跳转”行为只会触发相匹配的`<Route>`对应的页面内容更新，而不会刷新整个页面。

`<Link>`做了3件事情:

- 有onclick那就执行onclick
- click的时候阻止a标签默认事件
- 根据跳转href(即是to)，用history (web前端路由两种方式之一，history & hash)跳转，此时只是链接变了，并没有刷新页面。而`<a>`标签就是普通的超链接了，用于从当前页面跳转到href指向的另一 个页面(非锚点情况)。

a标签默认事件禁掉之后做了什么才实现了跳转?

```javascript
let domArr = document.getElementsByTagName('a')
[...domArr].forEach(item=>{
    item.addEventListener('click',function () {
        location.href = this.href
    })
})
```

### 5. React-Router如何获取URL的参数和历史对象？

**（1）获取URL的参数**

- **通过get传值**

路由配置还是普通的配置，如：`'admin'`，传参方式如：`'admin?id='1111''`。通过`this.props.location.search`获取url获取到一个字符串`'?id='1111'` 可以用url，qs，querystring，浏览器提供的api URLSearchParams对象或者自己封装的方法去解析出id的值。

- **通过动态路由传值**

路由需要配置成动态路由：如`path='/admin/:id'`，传参方式，如`'admin/111'`。通过`this.props.match.params.id` 取得url中的动态路由id部分的值，除此之外还可以通过`useParams（Hooks）`来获取

- **通过query或state传值**

传参方式如：在Link组件的to属性中可以传递对象`{pathname:'/admin',query:'111',state:'111'};`。通过`this.props.location.state`或`this.props.location.query`来获取即可，传递的参数可以是对象、数组等，但是存在缺点就是只要刷新页面，参数就会丢失。

**（2）获取历史对象**

1、使用this.props.history获取历史对象

```javascript
let history = this.props.history;
```

2、 16.8版本以上可以使用 React Router中提供的Hooks

```javascript
import { useHistory } from "react-router-dom";
let history = useHistory();
```





### 6. React-Router 4怎么在路由变化时重新渲染同一个组件？

当路由变化时，即组件的props发生了变化，会调用componentWillReceiveProps等生命周期钩子。那需要做的只是： 当路由改变时，根据路由，也去请求数据：

```javascript
class NewsList extends Component {
  componentDidMount () {
     this.fetchData(this.props.location);
  }
  
  fetchData(location) {
    const type = location.pathname.replace('/', '') || 'top'
    this.props.dispatch(fetchListData(type))
  }
  componentWillReceiveProps(nextProps) {
     if (nextProps.location.pathname != this.props.location.pathname) {
         this.fetchData(nextProps.location);
     } 
  }
  render () {
    ...
  }
}
```

利用生命周期componentWillReceiveProps，进行重新render的预处理操作。

### 7. React-Router的路由有几种模式？

React-Router 支持使用 hash（对应 HashRouter）和 browser（对应 BrowserRouter） 两种路由规则， react-router-dom 提供了 BrowserRouter 和 HashRouter 两个组件来实现应用的 UI 和 URL 同步：

- BrowserRouter 创建的 URL 格式：[xxx.com/path](https://link.juejin.cn?target=http%3A%2F%2Fxxx.com%2Fpath)
- HashRouter 创建的 URL 格式：[xxx.com/#/path](https://link.juejin.cn?target=http%3A%2F%2Fxxx.com%2F%23%2Fpath)

**（1）BrowserRouter**

它使用 HTML5 提供的 history API（pushState、replaceState 和 popstate 事件）来保持 UI 和 URL 的同步。由此可以看出，**BrowserRouter 是使用 HTML 5 的 history API 来控制路由跳转的：**

```javascript
<BrowserRouter
    basename={string}
    forceRefresh={bool}
    getUserConfirmation={func}
    keyLength={number}
/>
复制代码
```

**其中的属性如下：**

- basename 所有路由的基准 URL。basename 的正确格式是前面有一个前导斜杠，但不能有尾部斜杠；

```javascript
<BrowserRouter basename="/calendar">
    <Link to="/today" />
</BrowserRouter>
复制代码
```

等同于

```javascript
<a href="/calendar/today" />
复制代码
```

- forceRefresh 如果为 true，在导航的过程中整个页面将会刷新。一般情况下，只有在不支持 HTML5 history API 的浏览器中使用此功能；
- getUserConfirmation 用于确认导航的函数，默认使用 window.confirm。例如，当从 /a 导航至 /b 时，会使用默认的 confirm 函数弹出一个提示，用户点击确定后才进行导航，否则不做任何处理；

```javascript
// 这是默认的确认函数
const getConfirmation = (message, callback) => {
  const allowTransition = window.confirm(message);
  callback(allowTransition);
}
<BrowserRouter getUserConfirmation={getConfirmation} />
复制代码
```

> 需要配合`<Prompt>` 一起使用。

- KeyLength 用来设置 Location.Key 的长度。

**（2）HashRouter**

使用 URL 的 hash 部分（即 window.location.hash）来保持 UI 和 URL 的同步。由此可以看出，**HashRouter 是通过 URL 的 hash 属性来控制路由跳转的：**

```javascript
<HashRouter
    basename={string}
    getUserConfirmation={func}
    hashType={string}  
/>
复制代码
```

**其参数如下**：

- basename, getUserConfirmation 和 `BrowserRouter` 功能一样；
- hashType window.location.hash 使用的 hash 类型，有如下几种：
  - slash - 后面跟一个斜杠，例如 #/ 和 #/sunshine/lollipops；
  - noslash - 后面没有斜杠，例如 # 和 #sunshine/lollipops；
  - hashbang - Google 风格的 ajax crawlable，例如 #!/ 和 #!/sunshine/lollipops。

### 8. React-Router 4的Switch有什么用？

Switch 通常被用来包裹 Route，用于渲染与路径匹配的第一个子 `<Route>` 或 `<Redirect>`，它里面不能放其他元素。

假如不加 `<Switch>` ：

```javascript
import { Route } from 'react-router-dom'

<Route path="/" component={Home}></Route>
<Route path="/login" component={Login}></Route>
```

Route 组件的 path 属性用于匹配路径，因为需要匹配 `/` 到 `Home`，匹配 `/login` 到 `Login`，所以需要两个 Route，但是不能这么写。这样写的话，当 URL 的 path 为 “/login” 时，`<Route path="/" />`和`<Route path="/login" />` 都会被匹配，因此页面会展示 Home 和 Login 两个组件。这时就需要借助 `<Switch>` 来做到只显示一个匹配组件：

```javascript
import { Switch, Route} from 'react-router-dom'
    
<Switch>
    <Route path="/" component={Home}></Route>
    <Route path="/login" component={Login}></Route>
</Switch>
复制代码
```

此时，再访问 “/login” 路径时，却只显示了 Home 组件。这是就用到了exact属性，它的作用就是精确匹配路径，经常与`<Switch>` 联合使用。只有当 URL 和该 `<Route>` 的 path 属性完全一致的情况下才能匹配上：

```javascript
import { Switch, Route} from 'react-router-dom'
   
<Switch>
   <Route exact path="/" component={Home}></Route>
   <Route exact path="/login" component={Login}></Route>
</Switch>
```

