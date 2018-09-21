---
title: 初识react(五) 零配置数据流解决方案 dva
date: 2018-08-24 19:17:31
tags: react
---


![](https://user-gold-cdn.xitu.io/2018/9/20/165f51af2b41ad27?w=567&h=167&f=png&s=4664)
#### 回顾
- [初识react(一) 揭开jsx语法和虚拟DOM面纱](https://juejin.im/post/5b4ee916f265da0f563dd184)
- [初识react(二) 实现一个简版的html+redux.js的demo](https://juejin.im/post/5b83be86f265da432a6ae5df)
- [初识react(三)在 react中使用redux来实现简版计数器](https://juejin.im/post/5ba06cb6e51d450e9e440032)
- [初识react(四) react中异步解决方案之 redux-saga](https://juejin.im/post/5ba1bb506fb9a05d2d0218a5)
> 纠正下，零配置是由[umi](https://umijs.org/zh/guide/)帮我们实现的，不是dva帮我们做的，但是dva-cli在下一个版本(即dva-cli 1.0)已经内置了umi，简直开发者福音,有兴趣朋友可以体验下最新版本。

```
npm i dva-cli@next -g  //安装下一代dva-cli，内置umi
dva new dvaTest
cd dvaTest
npm start
```
> 我们这里还是从最基本的dva开始讲起，了解流程。**重点：dva并没有发明新的概念，全都是以前提到的。只是进行了一层封装**，对redux、saga中的概念很清楚的话，dva就是白给你的，没有难点，不会来找我。

#### 简介
- 基于 redux、redux-saga 和 react-router 的轻量级前端框架。
- dva是基于react+redux最佳实践上实现的封装方案，简化了redux和redux-saga使用上的诸多繁琐操作

#### 数据流向
- 数据的改变发生通常是通过：
    - 用户交互行为（用户点击按钮等）
    - 浏览器行为（如路由跳转等）触发的
- 当此类行为会改变数据的时候可以通过 dispatch 发起一个 action，如果是同步行为会直接通过 Reducers 改变 State ，如果是异步行为（副作用）会先触发 Effects 然后流向 Reducers 最终改变 State。
![](https://user-gold-cdn.xitu.io/2018/9/20/165f596c998fe3d0?w=1614&h=508&f=png&s=115320)

#### 实现的demo效果
> 由于dva比较简单，没有什么新概念用例子讲解会更明白。最后要实现一个异步获取数据 num，然后点击计数器 + num的效果
- 目录结构

![](https://user-gold-cdn.xitu.io/2018/9/20/165f620bfdc2c75c?w=677&h=181&f=png&s=4595)

#### 1、主入口文件

```
import React from 'react';
import dva from 'dva';
import Counter from './Counter';
//dva是一个函数，通过执行它可以拿到一个app对象
let app = dva();
//一个模板就是一个状态，然后把reducer和状态 写在一起了，
//添加一个模块
app.model({
   xxxx
});
//参数是一个函数，此应用本身就是要渲染函数的返回值
app.router(() => <Counter />);

//本质是启动应用，就是通过app.router获取组件，并且通过ReactDOM渲染到容器内容
app.start('#root');

```
> 以上是最基本的dva的主入口文件，简单的3个API，app.model、app.router、app.start，就已经讲react、redux-router、redux、redux-saga整合一起，简直开发者福音。我们讲解下怎么用

- dva是一个函数，通过执行它可以拿到一个app对象
- app.model()添加一个模块，下面重点讲解
- app.router()接受函数，然后渲染函数返回值
- app.start('#root')，通过app.router获取组件，然后通过ReactDom渲染到容器

##### 1.1、app.model()用法
- 接受一个对象，把state、reducers、effects全部写在这，便于维护。

```
app.model({
    //命名空间。因为一个应用会有很多个模型,每个模型要有一个名字
    namespace: 'counter',
    //此命名空间的默认状态
    state: { current: 0, highest: 0 },
    //它是用来接收action,修改仓库状态的
    reducers: {
        save(state, action) {
            return { current:state.current+action.payload  };
        }
    }
});

```
> 看见这些名词应该很熟悉吧

- namespace命名空间，我们需要给模型一个名字
- state=>状态，就是redux中的状态
- reducers=>处理器，就是redux中的处理器
> **在强调遍,dva没有发明新的概念，只是进行了一层封装。让状态更利于维护**
- [redux中概念还不清楚看这里](https://juejin.im/post/5b83be86f265da432a6ae5df)

#### 2、编写Counter.js组件
```
import React from 'react';
import { connect } from 'dva';
class Counter extends React.Component {
    render() {
        return (
            <div className="container">
                <div className="current">
                    当前记录:{this.props.current}
                </div>
                <div className="addButton">
                    <button onClick={() => this.props.dispatch({ type: 'counter/save'，payload:2 })}>+</button>
                </div>
            </div>
        )
    }
}
export default connect(
    state => {
        return state.counter;
    }
)(Counter);

```
> 不过多解释，有2个地方需要注意:
- 组件内部派发动作时，type:'counter/add'，前面多了counter(命名空间)
- connect时的状态是总的状态，需要制定下需要counter的状态

![](https://user-gold-cdn.xitu.io/2018/9/21/165fa2471920c491?w=1213&h=347&f=png&s=12221)

> 目前为止，dva流程已经跑通了，是不是很简单，我们测试下是否能点击加2

![](https://user-gold-cdn.xitu.io/2018/9/20/165f64798ddee537?w=252&h=449&f=gif&s=470662)

> 完美实现，说好的异步呢，接下来我们用express编写一个简单接口

#### 3、编写服务端接口 server.js
> 我们用express编写简单接口,不讲解express用法。 [express直通车](http://www.expressjs.com.cn/)

```
let express = require('express');
let cors = require('cors'); //解决跨域的包
let app = express();
app.use(cors()); //使用中间件cors
app.get('/amount', function (req, res) {
    res.send({ amount: 5 });
});
app.listen(3000);
```
- 接下来启动服务,看下效果
![](https://user-gold-cdn.xitu.io/2018/9/20/165f64f7a56fa9c9?w=966&h=238&f=png&s=17318)

#### 4、客户端发请求获取数据
> 由于案例比较简单，都写在了src/index.js中

```
function getAmount() {
    return fetch('http://localhost:3000/amount', {
        headers: {
            "Accept": "application/json"
        }
    }).then(res => res.json());
}
```
#### 5、在app.model中添加effects(副作用) (就是redux-saga中的effects)

```
  effects: {
        //表示这是一个generator effect=redux-saga/effects
        *add(action, { call, put }) {
            let { num } = yield call(getAmount); 
            yield put({ type: 'save', payload: num });
        }
    },
```
> 先异步获取数据，然后再派发动作修改状态，接着刷新视图

#### 6、对应的组件新增一个异步记数的按钮

```
<button onClick={() => this.props.dispatch({ type: 'counter/save',payload:2 })}>同步加2</button>
<button onClick={() => this.props.dispatch({ type: 'counter/add' })}>异步记数</button>
```
- 增加了一个异步计数按钮，会派发add动作类型。
- add类型被effects(副作用)中的add监听到，执行 getAmount()异步获取数据
- 拿到数据后派发save动作，被reducers处理
- 页面刷新

> 测试结果

![](https://user-gold-cdn.xitu.io/2018/9/20/165f6a9a9dca3a90?w=252&h=449&f=gif&s=883376)

#### 完结
> dva 简化了redux和redux-saga使用上的诸多繁琐操作,便于我们开发，可维护性也更高，配合umi使用，号称零配置，下篇文章会讲解dva+umi使用

- 如果对您有帮助，点个喜欢再走呗
- [更多优质文章参考](https://chaiguanpeng.github.io/)
- [redux所有源码解析戳这里](https://github.com/chaiguanpeng/react-code-analysis)


