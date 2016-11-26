[上一篇](https://github.com/ingf/ingf.github.io/issues/2) 主要介绍了 React，今天主要介绍一下 Redux，React + Redux 非常精炼，良好运用将发挥出极强劲的生产力。他们直接没有直接的联系，但是他们搭配在一起真的很般配，先稍微回顾一下 React。

## 对 React 的理解
- 理想的 UI 编程模型

  React 大胆的跳出了 Web 的 DOM 实现约束，实现了高效的渲染机制，树形控制描述 UI、统一事件机制、单向数据流。
- JSX

  JSX 是 React 中一颗闪耀的星星，她一个非常好的工程化手段，JSX 允许我们将标签写入 JavaScript 文件中，其实这样做更内聚。你第一眼看到她时，觉得她并不美，但是你多花五分钟，端详片刻，你就能体会到，而且她还有良好的开发体验，能在编译时快速清晰报错，指定错误代码行号，方便调试。
  说到 JSX，一般都会和模板进行比较，模板？套用一句功夫熊猫里面的台词，这真是太遗憾了，我想对模板的粉丝说一个不幸的消息，JSX 天生具有模板无法触及的优势。
  **模板是把 “JavaScript” 放入 HTML 里面，而 JSX 是把 “HTML” 放入 JavaScript 里面。**
  但因为 JavaScript 远比 HTML 要强大，在 JSX 里面可以使用到强大的 JavaScript，你可以获得 JavaScript 编程语言的能力和工具链来描述页面。因此，增强 JavaScript 使其支持HTML 要比增强 HTML 使其支持逻辑要合理的多。
  当然 JSX 也并非十全十美，她也有一些陷阱，也不能很好的处理 if-else，这些糟粕不足以盖住 JSX 的光芒。
## Redux
如果对于 Redux 比较熟悉了，可以略过这部分直接跳到最后的 **State 数据结构的设计** 这一节，这部分是我们团队在开发了好几个大型项目后总结出来的。

#### Redux = reduce + Flux

我们看一下 Javascript 中 `Array.prototype.reduce` 的用法，对数组中的所有元素调用指定的回调函数，该回调函数的返回值为累积结果，并且此返回值在下一次调用该回调函数时作为参数提供：

```
const initState = ''
const actions = ['Learn about actions, ','Learn about reducers, ', 'Learn about store']
const newState = actions.reduce((prevState, action)=> { return prevState + action }, initState)
```

这就是 Redux 的核心所在，`State` 就是应用程序的数据，给定 `initState` 之后，随着 `action` 的值不断传入给计算函数，得到新的 `State`。`Array.prototype.reduce` 的第一个参数是一个函数：`(prevState, actions)=> { return prevState + actions }`，这个计算函数被称之为 `Reducer`。

看到这里的童鞋，大部分都是 Web 前端工程师，我们每天的工作就是构建很多网页，呈现给用户。恩，对，我们每天构建了很多网页，那么我想问一个问题，我们每天构建了这么多网页，那么对于我们来说，网页的本质是什么？

网页的本质是存储在服务器上的一个文件，开玩笑，我们要说的当然不是这个。在我们 Web 前端工程师看来，网页的本质是将数据渲染出来，呈现给用户。这里有两个关键字，一是数据，二是渲染，当数据发生改变的时候，网页需要重新渲染。网页渲染和重新渲染，上面已经讲过了，就是 React，数据（这里的数据和上面提到的 `State`是同一个东西，因为 `State` 是 React 中的概念，所以以下将会使用 `State` 这个概念）和 数据改变的逻辑就是我们的主角 -- Redux。

Redux 约定：

> 整个应用的 State 被储存在一棵 Object tree 中，并且这个 Object tree 只存在于唯一一个 Store 中。

比如说，我们有这么一个 State：

```
[{
  todo: 'Learn about actions'
}]
```

这个 State 只有一条记录，React 将他渲染成

```
<span> Learn about actions </span>
```

我们把 State 修改以后

```
[{
  todo: 'Learn about actions'
},{
  todo: 'Learn about reducers'
}]
```

那么渲染也会发生变化

```
<span> Learn about actions </span>
<span> Learn about reducers </span>
```

这就是 Redux 中最基本的概念：
页面中的所有状态，都应该以这种状态树的形式来描述，页面上的任何变化，都应该先去改变这个状态树，然后再渲染到页面上。

下面就可以解释 Redux 几个核心概念了。
### Action

Action 是把数据从应用传到 Store 的有效载荷，它是 Store 数据的唯一来源。通俗的说，就是描述“发生了什么事情”。
那如何来描述上面的例子中，我们在 todo list “Learn about actions” 中新加入一条记录 “Learn about reducers”呢

```
export function addTodo(text) {
  return {
    type: ‘ADD_TODO',
    text
  }
}
```

这个函数会返回一个 Action 对象，Action 本质上是 JavaScript 普通对象，这个对象描述“发生了什么事情”。Redux 约定，Action 内使用一个字符串类型的 type 字段来表示将要执行的动作，除此以外，还可以携带动作所需要的数据，随后这个对象会被传入到 Reducer 中。
### Reducer

Action 只是描述了有事情发生了这一事实，并没有指明应用如何更新 State。而这正是 reducer 要做的事情。reducer 就是一个函数，接收旧的 State 和 Action，返回新的 State。

```
(prevState, action) => newState
```

我们现在用这个公式来解决上面的 Action。

```
const initialState = [{
  text: 'Learn about actions'
}]

export default function reducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return [{
          id: state.reduce((maxId, todo) => Math.max(todo.id, maxId), -1) + 1,
          completed: false,
          text: action.text
        },
        ...state
      ]
  }
}
```
### Sotre

我们学会了使用 Action 来描述“发生了什么”，和使用 reducers 来根据 Action 更新 State 的用法。

Store 就是把它们联系到一起的对象。Store 有以下职责：
- 维持应用的 State
- 提供 getState() 方法获取 State
- 提供 dispatch(action) 方法更新 State
- 通过 subscribe(listener) 注册监听器

再次强调一下 Redux 应用只有一个单一的 Store。

根据已有的 reducer 来创建 Store 是非常容易的。

```
import { createStore } from 'redux'
import todoApp from './reducers'
let store = createStore(todoApp)
```
### 发起 Actions

Redux 总结起来就一句话

> 将 **动作(Action)** 通过 **状态转换函数(Reducer)**，set 到一个统一的地方(**Store**)，然后 UI 从 Store 中获取数据。

接下来，我们验证一下我们的逻辑。

```
store.dispatch(actions.addTodo('Learn about actions'))    // 添加一个 todo
store.dispatch(actions.addTodo('Learn about reducers'))   // 添加一个 todo
store.dispatch(actions.addTodo('Learn about store'))      // 添加一个 todo
store.dispatch(actions.completeTodo(0))                   // 完成第零个 todo
store.dispatch(actions.completeTodo(1))                   // 完成第一个 todo
store.dispatch(actions.clearCompleted())                  // 清除已经完成的 todo
```

![image](https://cloud.githubusercontent.com/assets/3368034/13290582/267cb9fa-db4f-11e5-89b3-227ca6b42c8f.png)

简直没有比这再清晰的了，你甚至都不用阅读我的源码，只需要看一下这个 Action 列表（每个 Action 日志中会打印出初始状态、执行动作和执行后的状态），就知道业务逻辑是怎样执行的，也不会出现 MVC 中一个 Model 更新了以后不知道哪些 View 会随之更新的情况了。这正是 Redux 的一个很大的优点 ---- 可预测性。因此，我们清楚的知道发生了什么改变（Action），改变之后的数据是什么样的（State），以及发生了哪些改变（Action 记录）。

这段代码可以在所有 JavaScript 环境下执行，这意味我们可以进行业务逻辑的单元测试，也意味着这套业务逻辑可以用于 Web，用于 iOS、Android、tvOS...
![screenshot 2016-04-17 20 17 19-min](https://cloud.githubusercontent.com/assets/3368034/14586925/74613fce-04da-11e6-9e8b-7266392236d4.png)

Redux 与 React 没有直接联系，Redux 用于管理 State，与具体的 UI 框架无关，不过官方和社区提供了很多库，来绑定 Redux 和其他 UI 框架，比如 React、Angular、Vue 等等。
### 搭配 React

为了在 React 中使用 Redux，官方提供的一个库 react-redux，用来结合 Redux 和 React 的模块。react-redux 提供了两个接口 Provider、connect。

首先，我们需要获取 react-redux 提供的 Provider，并且在渲染之前将根组件包装进 `<Provider>`。

```
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import App from './web/containers/App'
import configureStore from 'app/store/configureStore'

const store = configureStore()

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

接下来，我们想要通过 react-redux 提供的 `connect()` 方法将包装好的组件连接到 Redux。

```
import React, { Component, PropTypes } from 'react'
import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'
import * as TodoActions from 'app/actions'

class App extends Component {
  render() {}
}

export default connect(
  (state) => {
    return {
      todos: state.todos
    }
  },
  (dispatch) => {
    return {
      actions: bindActionCreators(TodoActions, dispatch)
    }
  }
)(App)
```


## State 数据结构的设计

第一次在项目中使用 Redux 的时候，对于 State 基本没有设计，所以后期基本是推倒重来，当时，我们的内心是奔溃的。
第二个项目的时候，我们按照页面结构来组织 State，刚开始很好，但是到后来，就会出现这样一个情况，在某个页面需要到其他页面去获取 State，这样虽然可行，但不是好的实现方式，我对于这种做法一直耿耿于怀。
所以，对于 State，我们一直在思考，回归到网页的本质，我们有数据，然后去渲染。我们再回过头来看看数据，应用包含以下数据：
- 服务器响应
- 缓存数据
- 本地生成尚未持久化到服务器的数据
- 也包括 UI 状态，如激活的路由，被选中的标签，是否显示加载动效或者分页器

对数据进行一个分类处理，可以分为 **业务逻辑数据** 和 **页面 UI 数据**。

举个栗子，对于电影票应用而言，业务逻辑数据包含，正在热映列表、即将上映列表、电影详情、影院列表、影院详情、座位信息、支付信息、订单中心、用户中心等等，如果按照业务逻辑来组织 State，那么就可以设计如下的数据结构：

```
let state = {
    movie: {
        hot: [],    // 正在热映
        coming: [], // 即将上映
        detail: {}, // 当前电影详情
    },
    cinema: {
        list: {},   // 影院列表
        detail: {}, // 当前影院详情
        scheds: [], // 影院排期
    },
    seat: {
        seats: {},  // 作为信息
        locked: {}, // 已锁座位
    },
    payment: {},
    order: {
        list: [],
        detail: {},
    },
    routing: {},
    pages:{},
}
```

当然，我并没有列全，这只是个参考，这样，所有的数据结构一目了然，方便理解和记忆。所有的数据存储好了以后，所有的 page 都是从这个 state 里面获取数据，如果使用传统的 select 来计算，当页面比较大的时候，会存在性能问题，所以出现了 Reselect。

Reselect 库可以创建可记忆的(Memoized)、可组合的 selector 函数。Reselect selectors 可以用来高效地计算 Redux store 里的衍生数据。

所以，我们在 state 和 page 中加了一层 selector。在 selector 里面在组装 page 需要的数据以及数据的计算，这个数据可能会来自多个业务 model。

比如说，在选座页会呈现出影片的信息、影院的信息、座位图的信息，所以就会从三个业务 model 中获取数据(调用三次 action)，然后计算，传递给选座页使用。

这只是一种方式，当然业务逻辑不同，数据格式和结构也会多种多样，找到适合自己的就好。

### 并行开发

定义好 State，团队可以按照统一的 State 数据结构开发，各自相对独立，互不干扰。
## 不足之处
- 对从 OOP 开发转过来的程序猿来说，函数式编程的概念接受起来需要一点门槛。
- JavaScript 对不变对象的支持并不是特别的友好，无论是引入 immutable.js 还是 ES6 的解构语法糖有时候都觉得 Reducer 里的代码读起来有些费力，特别是对刚接触 ES6 的同学来说。



以上，源码在[这里](https://github.com/ingf/caption)
## 参考备注
- Redux：http://redux.js.org/
- thinking-in-react：https://facebook.github.io/react/docs/thinking-in-react.html
- Angular 2 versus React: There Will Be Blood：https://medium.freecodecamp.com/angular-2-versus-react-there-will-be-blood-66595faafd51#.3ummmr705
- React’s JSX: The Other Side of the Coin：https://medium.com/@housecor/react-s-jsx-the-other-side-of-the-coin-2ace7ab62b98#.f9cmo8qrd
