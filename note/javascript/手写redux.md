# 手写redux

首先需要讲一些概念。

- 纯函数

> 一个函数的返回结果只依赖于它的参数，并且在执行过程里面没有副作用，我们就把这个函数叫做纯函数。
> 特点: 
  1. 函数的返回结果只依赖于它的参数。
  2. 函数执行过程里面没有副作用。

- React.js 特性 —— context

> 组件间 共享状态 属性
> 只能传入子组件
> 父组件不能读取该属性

- 高阶组件

> 高阶组件是一个函数（而不是组件），它接受一个组件作为参数，返回一个新的组件

- 共享结构的对象

> 就是类似把 obj 所有的属性都复制到 obj2 里面，相当于对象的浅复制，obj 里面的内容和 obj2 是完全一样的，但是却是两个不同的对象。除了浅复制对象，还可以覆盖、拓展对象属性
> 例：  ``` const obj = { a: 1, b: 2}
const obj2 = { ...obj, b: 3, c: 4} // => { a: 1, b: 3, c: 4 }，覆盖了 b，新增了 c```


## 简介

在一个正常的 react app中需要 组件 也需要组件间交互的数据的store。在这个app中我们用provider 容器包裹 组件，用context 中包围store, 在组件间共享。每个组件都做纯组件化，不包含逻辑，只根据props 渲染。 这就是redux 的设计模式。


套路就是

```javascript
// 定一个 reducer
function reducer (state, action) {
  /* 初始化 state 和 switch case */
}

// 生成 store
const store = createStore(reducer)

// 监听数据变化重新渲染页面
store.subscribe(() => renderApp(store.getState()))

// 首次渲染页面
renderApp(store.getState()) 

// 后面可以随意 dispatch 了，页面自动更新
store.dispatch(...)
```


## 组件

组件就不多说了，主要 Smart 组件 vs Dumb 组件。

Dumb 组件 不包含所有逻辑
Smart 组件包含逻辑和从context 中获取相应的数据

## store

它接受的参数叫 reducer，reducer 是一个函数, 而且是纯函数（Pure Function）
```javascript
function createStore (reducer) {
  let state = null
  const listeners = []
  // 观察着模式 发布和订阅
  // 根据数据是否更新重新渲染相应的足迹
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    state = reducer(state, action)
    listeners.forEach((listener) => listener())
  }
  dispatch({}) // 初始化 state
  return { getState, dispatch, subscribe }
}
```

## reducer
reducer接受两个参数，一个是 state，一个是 action。
如果没有传入 state 或者 state 是 null，那么它就会返回一个初始化的数据。如果有传入 state 的话，就会根据 action 来“修改“数据，但其实它没有、也规定不能修改 state，而是要通过上节所说的把修改路径的对象都复制一遍，然后产生一个新的对象返回。如果它不能识别你的 action，它就不会产生新的数据，而是（在 default 内部）把 state 原封不动地返回。

> reducer 是不允许有副作用的。你不能在里面操作 DOM，也不能发 Ajax 请求，更不能直接修改 state，它要做的仅仅是 —— 初始化和计算新的 state。

```javascript

function themeReducer (state, action) {
  if (!state) return {
    themeName: 'Red Theme',
    themeColor: 'red'
  }
  switch (action.type) {
    case 'UPATE_THEME_NAME':
      return { ...state, themeName: action.themeName }
    case 'UPATE_THEME_COLOR':
      return { ...state, themeColor: action.themeColor }
    default:
      return state
  }
}

const store = createStore(themeReducer)
...
```

[Github](https://github.com/lirawx/Crazy_FED/tree/master/react/make-react-redux)
部分代码
react-redux.js

```javascript

import React, { Component } from 'react'
import PropTypes from 'prop-types'

export const connect = (mapStateToProps, mapDispatchToProps) => (WrappedComponent) => {
  class Connect extends Component {
    static contextTypes = {
      store: PropTypes.object
    }

    constructor () {
      super()
      this.state = {
        allProps: {}
      }
    }

    componentWillMount () {
      const { store } = this.context
      this._updateProps()
      store.subscribe(() => this._updateProps())
    }

    _updateProps () {
      const { store } = this.context
      let stateProps = mapStateToProps
        ? mapStateToProps(store.getState(), this.props)
        : {} // 防止 mapStateToProps 没有传入
      let dispatchProps = mapDispatchToProps
        ? mapDispatchToProps(store.dispatch, this.props)
        : {} // 防止 mapDispatchToProps 没有传入
      this.setState({
        allProps: {
          ...stateProps,
          ...dispatchProps,
          ...this.props
        }
      })
    }

    render () {
      return <WrappedComponent {...this.state.allProps} />
    }
  }
  return Connect
}


export class Provider extends Component {
  static propTypes = {
    store: PropTypes.object,
    children: PropTypes.any
  }

  static childContextTypes = {
    store: PropTypes.object
  }

  getChildContext () {
    return {
      store: this.props.store
    }
  }

  render () {
    return (
      <div>{this.props.children}</div>
    )
  }
}
```

index.js

```javascript


import React, { Component } from 'react'
import PropTypes from 'prop-types'
import ReactDOM from 'react-dom'
import Header from './Header'
import Content from './Content'
import './index.css'

import { Provider } from './react-redux'

function createStore (reducer) {
  let state = null
  const listeners = []
  const subscribe = (listener) => listeners.push(listener)
  const getState = () => state
  const dispatch = (action) => {
    state = reducer(state, action)
    listeners.forEach((listener) => listener())
  }
  dispatch({}) // 初始化 state
  return { getState, dispatch, subscribe }
}

const themeReducer = (state, action) => {
  if (!state) return {
    themeColor: 'red'
  }
  switch (action.type) {
    case 'CHANGE_COLOR':
      return { ...state, themeColor: action.themeColor }
    default:
      return state
  }
}

const store = createStore(themeReducer)


// 删除 Index 里面所有关于 context 的代码
class Index extends Component {
  render () {
    return (
      <div>
        <Header />
        <Content />
      </div>
    )
  }
}

// 把 Provider 作为组件树的根节点
ReactDOM.render(
  <Provider store={store}>
    <Index />
  </Provider>,
  document.getElementById('root')
)
```



