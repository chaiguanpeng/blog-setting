---
title: 初识react(二) 实现一个简版的redux
date: 2018-08-27 22:43:20
tags: js
---

### 前言
> 首先纠正个误区，redux可以配合jq等框架使用，并不单单局限于react。为了让所有人都看懂，我们这里先只实现一个最简单版本的 html+redux.js的使用。

### 为什么出现redux
> 随着单页面应用的复杂，多个没有联系的组件之间想要共享状态(state)很困难，Redux的出现解决了数据问题
![](https://user-gold-cdn.xitu.io/2018/8/27/1657b5cda1448bb7?w=661&h=334&f=png&s=45242)

### redux三大原则

#### 单一数据源
- 整个应用的state都被存储在一个仓库中，我们称之为store,整个应用只能有一个store。

#### 只读的state
- 唯一改变state的方法就是dispatch(action)，即派发动作。

#### 使用纯函数执行修改
- 为每个action用纯函数编写reducer来描述如何修改state
> 说了这么多，看不懂?没关系，就是有三个概念 state、reducer、action。下面我们一一讲解API

### 概念解析

#### 1、store 仓库
- redux提供了一个createStore函数，用来生成store
- store就是保存数据的地方，可以看成一个容器。整个应用只能有一个store

```
function createStore(reducer) { //将状态放到一个盒子里 别人改不了
    ...
}
let store = createStore(reducer)

```
#### 2、State 状态
> store对象包含所有数据。如果想得到某个时点数据,就要对Store生成快照。这种时间点的数据集合，就叫做State。 当前时刻的State，可以通过store.getState()拿到。

```
let state = store.getState()
```
#### 3、action 动作
- action必须是一个对象,type是必须的，用户可以派发动作来改变state.

```
let action = {
    type:"change_title_text",
    text:"改变标题了"
}
```
#### 4、store.dispatch(action) 派发动作
- store.dispatch()是发出action的唯一方法

```
let store = createStore(reducer);
store.dispatch(action)  //action=>{type:"change_title_text",text:"改变标题了"}
```
#### 5、Reducer 管理员，也可以称之为处理器
- Store收到派发的动作后即dispatch(action)，必须返回一个新的state，这样视图才会变化。
- 这种state的计算过程叫做Reducer，是一个纯函数，接受state和action作为参数，返回一个新的state

```
let reducer = function(state,action){
    return new_state;
}
```
> 废话了这么多，很庆幸把基本概念说完了，终于来个实战来看看具体怎么工作的。我们做一个最简单计算器，点击加1，来看看redux怎么工作的

### 计数器实现步骤(redux)

#### 1、声明一个初始化状态

```
let initState = {number:0}
```
#### 2、createStore 重点
- 创建仓库，保存状态，对外暴露当前状态=>getState 和如何更改状态的方法=>dispatch
```
let createStore=(reducer)=> { //将状态放到一个盒子里 别人改不了
    let state ;  //声明状态
    function dispatch(action) { //派发 参数是action动作,action是一个对象
       state= reducer(state,action); //调用写好的方法,这个方法会返回一个新的状态
    }
    dispatch({}); //内部需要先定义次
    let getState = ()=> JSON.parse(JSON.stringify(state)); //获取状态的方法 深拷贝
    return {
        getState,
        dispatch
    };
}

```
- 需要知道 action是一个对象=>{type:"add",count:5}，类型为add,每次点击加5
- 在创建仓库的时候,会默认先调用,dispatch({})，给state赋值默认状态
- 对外暴露 getState方法,用户可以 获取最新状态
- 对外暴露 dispatch方法，用户可以派发动作

> 当看不懂时,只要知道目的只有一个，就是在给state赋默认值。  先dispatch({})=>reducer(state,action)。就可以赋默认值拉，至于为什么，往下看

#### 3、reducer实现   
- 管理员，可以根据类型返回不同的状态
```
let reducer=(state=initState,action)=> { //管理员,负责如何更改状态的
    switch (action.type) { //更改状态 要有一个新的状态覆盖掉
        case "add":
            return {number:state.number+action.count}
    }
    return state;
};
```

> 好了，到目前为止，我们来梳理下思路
- 我们会这样调用  let store = createStore(reducer),这其中发生了什么，如何把初始状态赋值给state的
- 步骤dispatch({}) =>reducer(initState,action)=>state=initState

#### 4、渲染页面视图为初始状态

```
let store = createStore(reducer);
function render() {
    let content = document.querySelector('.content');
    content.innerHTML = store.getState().getState().number;
}
render();
```
- 将页面视图与store中的state进行绑定。看效果

![](https://user-gold-cdn.xitu.io/2018/8/27/1657bbd4a9de7a10?w=2128&h=955&f=png&s=62472)

> 目前为止，一切完美，那我们怎么点击按钮改变状态，只能通过store.dispatch()方法

#### 5、点击改变视图

```
btn.onclick = function () {
    store.dispatch({type:"add",count:5});
    render()
}
```
![](https://user-gold-cdn.xitu.io/2018/8/27/1657bc3df6414d2d?w=2126&h=1140&f=png&s=75997)

> 到目前为止，一个最简单的redux应用算实现了，其实redux还是比较简单的，重点是理解它概念，后续会讲解在react中如何使用redux

- [redux全部源码解析，可以参考我总结](https://github.com/chaiguanpeng/react-code-analysis)
