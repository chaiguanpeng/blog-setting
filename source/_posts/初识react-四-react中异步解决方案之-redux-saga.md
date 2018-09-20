---
title: 初识react(四) react中异步解决方案之 redux-saga
date: 2018-08-25 19:17:00
tags: react
---


![](https://user-gold-cdn.xitu.io/2018/9/19/165efc712821a836?w=800&h=167&f=png&s=21769)

### 回顾
- [初识react(一) 揭开jsx语法和虚拟DOM面纱](https://juejin.im/post/5b4ee916f265da0f563dd184)
- [初识react(二) 实现一个简版的html+redux.js的demo](https://juejin.im/post/5b83be86f265da432a6ae5df)
- [初识react(三)在 react中使用redux来实现简版计数器](https://juejin.im/post/5ba06cb6e51d450e9e440032)

> 今天demo是实现一个异步的计算器，探究redux-saga工作流程

### 简介
- redux-saga 是一个 redux 的中间件，而中间件的作用是为 redux 提供额外的功能。
- 由于在 reducers 中的所有操作都是同步的并且是纯粹的，即 reducer 都是纯函数，纯函数是指一个函数的返回结果只依赖于它的参数，并且在执行过程中不会对外部产生副作用，即给它传什么，就吐出什么。
- 但是在实际的应用开发中，我们希望做一些异步的（如Ajax请求）且不纯粹的操作（如改变外部的状态），这些在函数式编程范式中被称为“副作用”。
> redux-saga 就是用来处理上述副作用（异步任务）的一个中间件。它是一个接收事件，并可能触发新事件的过程管理者，为你的应用管理复杂的流程。

### redux-saga工作原理
- 对generator不了解的，看下[阮一峰 generator讲解](http://www.ruanyifeng.com/blog/2015/04/generator.html)
- sages 采用 Generator 函数来 yield Effects（包含指令的文本对象）。
- Generator 函数的作用是可以暂停执行，再次执行的时候从上次暂停的地方继续执行
- Effect 是一个简单的对象，该对象包含了一些给 middleware 解释执行的信息。
- 你可以通过使用 effects API 如 fork，call，take，put，cancel 等来创建 Effect。

###  redux-saga分类
- worker saga 做左右的工作，如调用API，进行异步请求，获取异步封装结果
- watcher saga 监听被dispatch的actions,当接受到action或者知道其被触发时，调用worker执行任务
- root saga 立即启动saga的唯一入口

> 基本介绍已经讲完了，当做完一个demo后，回头再看[redux-saga官网](https://redux-saga-in-chinese.js.org/)或者上面讲解，可能会有更深的体会

### 使用redux-saga实现一个异步计数器
> 由于目录结构跟上篇文章一样，在这里就只把变动的部分单独抽离出来讲解
- [先回顾下初识react(三)在 react中使用redux来实现简版计数器](https://juejin.im/post/5ba06cb6e51d450e9e440032?utm_source=gold_browser_extension)

#### 1、修改actions/counter.js
- 增加一个异步记数的动作类型。

```
import * as Types from "../action-types";
let actions ={
    add(num){
        return{type:Types.INCREMENT,count:num}
    },
    minus(num){
        return{type:Types.DECREMENT,count:num}
    },
    //增加了一个异步记数的类型，用于在counter.js中派发动作
    async(num){
        return {type:Types.ADD_ASYNC}
    }
};
export default actions;

```
#### 2、重点，在src目录下增加saga.js文件

```
//takeEvery=>负责监听  put=>派发动作   call=>告诉saga，执行delay，并传入1000作为参数
import {takeEvery,put,call} from "redux-saga/effects";
import * as Types from "./store/action-types";
const delay = ms=>new Promise((resolve,reject)=>{
    setTimeout(()=>{
        resolve()
    },ms)
})
//saga分为三类 1、rootsaga 2、监听saga 3、worker干活的saga
function* add() {
    yield call(delay,1000);
    //就是指挥saga中间件向仓库派发动作
    yield put({type:Types.INCREMENT,count:10});
}
function* watchAdd() {
    //监听派发给仓库的动作，如果动作类型匹配的话，会执行对应的监听生成器
    yield takeEvery(Types.ADD_ASYNC,add)
}
export default function* rootSaga() {
    yield watchAdd()

}
```
> 还记得上面说的,rudux-saga分类,root saga ->watcher saga -> worker saga。在这段代码中将会体现，代码我们从上往下看。


##### 2.1 saga工作流程(代码从下往上看)
- 默认导出了rootSaga,即saga的入口文件
- watcher saga  ->wactchAdd 负责监听派发的动作，如果动作类型匹配，执行对应的 worker saga
- 上面的worker saga指代的是 add函数

##### 2.2 redux-saga/effects 副作用
> 在实际的应用开发中，我们希望做一些异步的（如Ajax请求）且不纯粹的操作（如改变外部的状态），这些在函数式编程范式中被称为“副作用”。

- takeEvery=>负责监听，监听派发给仓库的动作，如果动作类型匹配的话，会执行对应的监听生成器->add
- put=>派发动作，可以理解为dispatch
- call=>告诉saga,执行delay函数，并把参数传过去。注意: delay函数必须返回promise

> 讲到这里，流程就说完了，接下来在store中执行rootSaga

#### 3、在store/index引入rootSaga

```
    import {createStore,applyMiddleware} from 'redux';
    import createSagaMiddleware from "redux-saga"; //引入redux-saga
    import  rootSaga from "../saga" //引入我们上面写好的rootSaga
    import reducer from "./reducers"
    let sagaMiddleware =createSagaMiddleware(); //执行得到saga中间件
    let store = createStore(reducer,applyMiddleware(sagaMiddleware)); //使用中间件
    sagaMiddleware.run(rootSaga); //开始执行rootSaga
    export  default  store;
```
> 对于redux中间件没有讲解，这部分内容涉及东西比较多，也不太好理解，写这个react系列目的是尽可能简单的让所有人理解，想看所有的redux源码解析，底部会留下所有总结的代码仓库。

#### 4、在counter.js组件中派发这个异步动作

- 代码跟上篇文章一模一样，只是增加了按钮实现异步操作
```
import React, {Component} from 'react';
import ReactDOM from 'react-dom';
import {connect} from "react-redux";
import actions from "../store/actions/counter"
class Counter extends Component {
    render() {
        console.log(this.props);
        return(
            <div>
                <h1>{this.props.number}</h1>
                <button onClick={()=>{this.props.add(5)}}>+</button>
                <button onClick={()=>{this.props.minus(1)}}>-</button>
                //增加异步操作
                <button onClick={()=>{this.props.async()}}>异步加10</button>
            </div>
        )
    }
}
export default connect(state=>({
    ...state
}),actions)(Counter)
```
> 终结，看效果。可以看出，点击后等待1s才加10。那我们就可以在call()中传入执行的异步函数(如ajax)来获取数据啦。我们这个例对应的delay函数

![](https://user-gold-cdn.xitu.io/2018/9/19/165f0feb3ce79301?w=252&h=449&f=gif&s=834695)

### 最后在梳理下整个过程
- 1、组件中调用了this.props.async(),返回的action对象=>{type:Types.ADD_ASYNC}会在connect方法中被派发
- 2、saga中takeEvery(Types.ADD_ASYNC,add),监听到动作的类型后，触发 worker saga =>add
- 3、worker saga中先 yield call(delay,1000); 执行delay方法，延时1s
- 4、yield put({type:Types.INCREMENT,count:10});  最后派发的还是INCREMENT的类型
- 5、接着被reducer处理，更新state
- 6、页面刷新

- [更多优质文章参考](https://chaiguanpeng.github.io/)
- [redux所有源码解析戳这里](https://github.com/chaiguanpeng/react-code-analysis)
 



