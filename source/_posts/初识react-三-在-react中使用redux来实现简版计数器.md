---
title: 初识react(三)在 react中使用redux来实现简版计数器
date: 2018-08-26 22:55:14
tags: react
---

- [初识react(二) 实现一个简版的html+redux.js的demo](https://juejin.im/post/5b83be86f265da432a6ae5df)
> 上篇文章已经把redux核心概念讲明白了，这里就不在废话啦，不了解的可以先去回顾下，接下来我们讲解下在react中如何使用redux，来实现一个简单计数器

### 先把目录结构搭好


![](https://user-gold-cdn.xitu.io/2018/9/18/165eab7d29254609?w=360&h=181&f=png&s=2476)

> 下面我们讲解下每个文件的作用，然后在一个一个实现
- actions->counter.js存放计数器的动作
- reducers->index.js是主入口文件，因为可能有好多个reducer。
- reducers->counter.js存放计数器的reducer
- action-types.js 存放宏，玩游戏的肯定知道，保存动作的类型
- store->index.js 是整个store对外暴露的入口文件
> 读过上篇文章后，对这种目录结构可能还不清楚具体的作用，但是对这些redux中概念肯定已经明白，下面我们一个一个文件开始讲解

### 一、编写store部分

#### 1、编写最简单的store/action-types.js文件
- 是一个宏文件，保存计数器动作的类型，说白了就是加、减操作。贴代码看的更具体

```
// action-types
export const INCREMENT = "INCREMENT";
export const DECREMENT = "DECREMENT";

```
> 宏文件不是必须的，但是它好处是结构清晰，一目了然可以看到计数器所有动作的类型

#### 2、编写reducers/counter.js文件
- 是一个计数器的reducer，reducers目录下可能有很多个reducer,我们先写一个

```
import * as Types from "../action-types"; //引入动作类型
let initState = {  //声明一个初始的状态
    number:0
};
function counter(state = initState, action) {   //接收state和action两个参数,并给state赋予初始值
    switch (action.type) {  //判断动作类型
        case Types.INCREMENT: //action类似这种结构{type:'INCREMENT'，count:5}
            return {number:state.number+action.count};
        case Types.DECREMENT:
            return {number:state.number-action.count};
    }
    return state    
}
export default counter

```
> 跟我们上篇文章写的reducer一模一样，只不过我们把它抽离出来，让结构清晰

#### 3、编写reducers/index.js文件
- 由于我们只有一个计数器的reducer，所以默认导出就可以。当有多个reducer，会在这个文件进行合并，后面再讲。

```
import counter from "./counter"; //默认导入reducer
export default counter      //默认导出
```
> 在啰嗦一句，主要是为了以后方便扩展多个reducer，所以才会有reducers/index.js这个主文件

#### 4、 编写actions/counters.js
- 把派发的动作抽离出来，用于来组件中调用

```
import * as Types from "../action-types"; //引入宏
let actions = {
    add(num){ //add方法会在组件内部调用，返回action对象
        return {type:Types.INCREMENT,count:num}
    },
    minus(num){
        return {type:Types.DECREMENT,count:num}
    }
};
export default actions
```
#### 5、编写store/index.js，仓库的主文件
- 这个文件主要用于导出store，提供给组件使用

```
import {createStore} from 'redux';
import reducer from "./reducers"
let store = createStore(reducer); //创建store
export default store;
```
> 截止到目前为止，store文件已经全部写完。下面我们开始写组件部分，让仓库中数据给组件使用

### 二、组件调用部分

#### 1、编写react的主入口文件，即 src/index.js
- 使用react-redux库，来实现store和组件之间的通信
- react-redux提供了2个核心API， Provider 提供  connect 链接
- Provider是一个组件，在react入口文件中用于提供store。
- connect含义是，在react组件内部连接store，进而实现组件与redux之间通信

> 怪我不能给大家讲明白，我们还是看代码逐句解释


```
import React from 'react';
import ReactDOM from 'react-dom';
import Counter from "./components/Counter";
import store from "./store";
//引入react-redux中的Provider组件，用于
import {Provider} from "react-redux";

ReactDOM.render(
    <Provider store = {store}>
        <div>
            <Counter />
        </div>
    </Provider>, document.getElementById('root'));

```

#### 2、编写Counter组件
- 组件中用到connect方法，实现组件与redux之间通信，connect方法接受2参数。connect(mapStateToProps,actions)(Counter)
- 把store/actions.js导出的对象绑定到组件的属性中，组件内部可以通过this.props拿到对应的actions

```
import React from "react";
import store from "../store";
import * as Types from "../store/action-types"
//生成action的对象的方法叫actionCreator
import actions from "../store/actions/counter";
import {connect} from "react-redux";
class Counter extends React.Component {
    render() {
        console.log(this.props);
        return (
            <div>
             <div>{this.props.number}</div>
                <button onClick={()=>{
                    this.props.add(5)
                }}>+</button>
                <button onClick={()=>{
                    this.props.minus(1)
                }}>-</button>
            </div>
            )
    }
}
let mapStateToProps = (state)=>{ //state代表的store.getState()
    return {...state}
};

export default connect(mapStateToProps,actions)(Counter)   

```
> 到此为止，我们基本实现一个计数器功能，先来测试下，然后在梳理下整个工作流程

![](https://user-gold-cdn.xitu.io/2018/9/18/165eb814d65aa598?w=382&h=399&f=png&s=7310)
> 测试点击增加记数功能

![](https://user-gold-cdn.xitu.io/2018/9/18/165eb83c1c9dee83?w=619&h=446&f=png&s=9138)

> 功能基本实现，可能对整个流程并不清楚怎么实现的，下面来梳理下整个工作流程

### react-redux整个流程分析
- 当点击按钮触发 this.props.add(5)，返回的action即{type:Types.INCREMENT,count:num}，会在connect内部被派发
- 派发动作后被reducer处理，然后返回新的状态
- 页面刷新
> 最后来张图结尾

![](https://user-gold-cdn.xitu.io/2018/9/18/165ebbbfd8154639?w=800&h=417&f=png&s=183854)

- [更多优质文章参考](https://chaiguanpeng.github.io/)
- [redux所有源码解析戳这里](https://github.com/chaiguanpeng/react-code-analysis)





