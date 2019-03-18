---
title: koa用法总结
cdn: 'header-on'
header-img: "/image/bg.jpg"
tags: node
---
#### 前言

koa是基于Node.js平台的下一代web开发框架,它体积小，扩展性强，给人一种干净利落的编程方式，且由express原班人马打造,国内很多互联网公司都在使用，因此有必要学习总结下。

#### 初出茅庐,来个hello word
- 按照惯例，先来个demo 输出hello world

```
let Koa = require('koa'); //引入koa
let app = new Koa();    //声明一个实例app
app.use(async (ctx,next)=>{   //  对于任何请求，app将调用该异步函数处理请求：
    ctx.body = "hello"
});
app.listen("3000");  //监听端口
```
- 其中参数ctx是由koa传入的封装了request和response的变量，我们可以通过它访问request和response。
- next是koa传入的将要处理的下一个异步函数。
- 由async标记的函数称为异步函数，在异步函数中，可以用await调用另一个异步函数，这个异步函数必须返回一个promise，[上篇文章写过promise用法和实现原理,不了解可以先去看看](https://juejin.im/post/5b2275a2e51d4558875321b7)。这两个关键字将在ES7中引入。

![](https://user-gold-cdn.xitu.io/2018/7/30/164e9d3d5ffc147d?w=1070&h=330&f=png&s=19267)
> 简捷的5行代码，帮我们开启了3000端口的服务

#### 深入理解Koa中间件之洋葱模型

学习Koa重点在于理解中间件实现原理，对后续引用第三方库中间件时候有更好了解。我们单独讲讲
- 先来段测试代码

```
let Koa = require('koa');
let app = new Koa();
app.use(async (ctx,next)=>{
    console.log(1);
   await next();
    console.log(2);
});
app.use(async (ctx,next)=>{
    console.log(3);
   await next();
    console.log(4);
});
app.listen("3000");
```
> 你可能对运行的结果会说 1234，其实不然，我们先来看下输出结果

![](https://user-gold-cdn.xitu.io/2018/7/30/164e9e63c68d0538?w=477&h=610&f=png&s=15495)


![](https://user-gold-cdn.xitu.io/2018/7/30/164e9ddde1bdf37f?w=295&h=300&f=jpeg&s=11603)

> 一脸懵逼1342,这是什么顺序，这就是我们要说的洋葱模型
- 中间件的执行很像一个洋葱，但并不是一层一层的执行，而是以next为分界，先执行本层中next以前的部分，当下一层中间件执行完后，再执行本层next以后的部分。
- 一个洋葱结构，从上往下一层一层进来，再从下往上一层一层回去,是不是有点感觉了。


![](https://user-gold-cdn.xitu.io/2018/7/30/164e9e8c42ddd370?w=478&h=435&f=png&s=153282)

#### 1、koa-router中间件

###### koa-router基础写法
- 为了讲的详细全面,把路由分为及部分来讲解，先来看最基本的路由怎么写

```
let Koa = require('koa');
let app = new Koa();
let Router = require('koa-router');
let router = new Router();
router.get('/',async (ctx,next)=>{
    ctx.body = 'hello people';
    await next()
});
router.get('/list',async (ctx,next)=>{
    ctx.body = 'list';
});
app.use(router.routes()); // 挂载
app.use(router.allowedMethods());//当请求数据的方法与设置的方法不一致，会报错。比如默认get请求获取，用post发请求会报错
app.listen(3000);
```

###### koa-router中嵌套路由写法

> 假如我们想为单个页面设置层级，/home是我们首页，再次基础上有/home/list 首页列表页 /home/todo 首页todo页。这时我们就需要用到嵌套路由，看看怎么用

```
const Koa = require('koa');
const app = new Koa();
const Router = require('koa-router');
//home的路由
let home = new Router();
home.get('/list',async(ctx)=>{
    ctx.body="Home list";
}).get('/todo',async(ctx)=>{
    ctx.body ='Home ToDo';
});
//page的路由
let page = new Router();
page.get('/list',async(ctx)=>{
    ctx.body="Page list";
}).get('/todo',async(ctx)=>{
    ctx.body ='Page todo';
});
//装载所有子路由
let router = new Router();
router.use('/home',home.routes(),home.allowedMethods());
router.use('/page',page.routes(),page.allowedMethods());
//加载路由中间件
app.use(router.routes()).use(router.allowedMethods());
app.listen(3000);

```
> 这样一来就实现嵌套路由的写法

![](https://user-gold-cdn.xitu.io/2018/7/30/164ea205fe0c6924?w=1265&h=261&f=png&s=56779)

###### koa-router参数的传递
- 1、通过/arcicle/id/name传参

```
let Koa = require('koa');
let app = new Koa();
let Router = require('koa-router');
let router = new Router();
//实现  /arcicle/id/name形式的传参
router.get('/acticle/:id/:name',(ctx,next)=>{
    ctx.body = ctx.params.id +"-"+ ctx.params.name;
});
app.use(router.routes());
app.listen(3000);
```
> 测试下，学过vue应该比较熟悉
![](https://user-gold-cdn.xitu.io/2018/7/30/164ea72319a995bc?w=1101&h=332&f=jpeg&s=37467)

- 2、通过/arcicle?id=1&name=cgp传参

```
const Koa = require('koa');
const Router = require('koa-router');
const app = new Koa();
const router = new Router();
router.get('/article', function (ctx, next) {
    ctx.body=ctx.query; //query方法实现json形式
});
app.use(router.routes())
app.listen(3000,()=>{
    console.log('starting at port 3000');
});
```

![](https://user-gold-cdn.xitu.io/2018/7/30/164ea790455a10a3?w=1102&h=301&f=png&s=20104)

#### 2、koa-bodyparse()中间件
- 用来解析请求体的中间件，比如获取post提交的表单数据，通过koa-bodyparse解析后就能获取到数据。看demo

```
let Koa = require('koa');
let bodyParser = require('koa-bod')
let app = new Koa();
app.use(bodyParser()); // 解析请求体的中间件
app.use(async (ctx, next) => {
    if (ctx.path === '/' && ctx.method === 'GET') {
        ctx.set('Content-Type', 'text/html;charset=utf8');
        ctx.body = `
        <form action="/" method="post">
            <input type="text" name="username" >
            <input type="text" name="password" >
            <input type="submit" >
        </form>
        `
    }
});
app.use(async (ctx, next) => {
    if (ctx.method === 'POST' && ctx.path === '/') {
        // 获取表单提交过来的数据
        ctx.body = ctx.request.body;
    }
});
app.listen(3000);

```
> 当post提交表单获得表单数据，测试下结果

![](https://user-gold-cdn.xitu.io/2018/7/30/164eb36e1db8c23a?w=1624&h=756&f=png&s=190898)

#### 3、koa-better-body中间件
- 是用来上传文件的中间件
- 由于老的中间件都是基于koa1版本的generate函数实现的，在koa2中我们需要用koa-convert，可以将他们转为基于Promise的中间件供Koa2使用
> 来个demo体验下，我们把本地的1.txt文件上传到upload文件夹中。 1.txt内容为123456789

```
let Koa = require('koa');
let app = new Koa();
let betterBody = require('koa-better-body'); // v1插件
let convert = require('koa-convert'); // 将1.0的中间件 转化成2.0中间件
app.use(convert(betterBody({
    uploadDir: __dirname //指定上传的目录 __dirname当前文件夹绝对路径
})))
app.use(async (ctx, next) => {
    if (ctx.path === '/' && ctx.method === 'GET') {
        ctx.set('Content-Type', 'text/html;charset=utf8');
        ctx.body = `
        <form action="/" method="post" enctype="multipart/form-data">
            <input type="text" name="username" autoComplete="off">
            <input type="text" name="password" autoComplete="off">
            <input type="file" name="avatar">
            <input type="submit" >
        </form>
        `
    } else {
        return next();
    }
});
app.use(async (ctx, next) => {
    if (ctx.method === 'POST' && ctx.path === '/') {
        // 获取表单提交过来的数据
        ctx.body = ctx.request.fields;
    }
});
app.listen(1000);
```
> 看下上传结果

![](https://user-gold-cdn.xitu.io/2018/7/30/164eb50a96487928?w=1869&h=706&f=png&s=57761)

> 内容也是正确的，我就不给大家展示拉

#### 4、kao-views中间件
- koa-views对需要进行视图模板渲染的应用是个不可缺少的中间件，支持ejs, nunjucks等众多模板引擎。
> 我们以ejs为例子

```
let Koa = require('koa');
let app = new Koa();
let views = require('koa-views');
app.use(views(__dirname,{
    extension:'ejs' //指定用ejs模板
}));
app.use(async (ctx,next)=>{
    // 渲染index.ejs
    await ctx.render('index',{name:'cgp',age:9,arr:[1,2,3]});
});
app.listen(3000);
```
- 我们来看下index.ejs模板是怎么写的。[ejs语法戳这里](https://ejs.bootcss.com/)
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <h1><%=name%></h1>
    <h1><%=age%></h1>
    <ul>
        <%arr.forEach(item=>{%>
             <li><%=item%></li>
        <%})%>
    </ul>
</body>
</html>

```

- 我们这里写的模板很简单，就是输出下name、age，然后循环下数组

![](https://user-gold-cdn.xitu.io/2018/7/30/164eb5d8613713a6?w=1077&h=346&f=png&s=18026)

#### 5、koa-static
- 一个用于生成静态服务的中间件 [上篇文章手写了cgp-server的模块](https://juejin.im/post/5b556aa8e51d451917171923)
```
let Koa = require('koa');
let server = require('koa-static');
let app = new Koa();
app.use(server(__dirname +'/public'));
app.listen(3000);
```

- 用法很简单，有兴趣可以看下我写的[静态服务器](https://juejin.im/post/5b556aa8e51d451917171923)

#### 6、koa自带cookie用法
>比如我们要存储用户名，保留用户登录状态时，会用到cookie。

###### 共两个方法
- ctx.cookies.get()
- ctx.cookies.set()
> 先来个demo测试，当输入/write写入cookie，当输入/read读到cookie
```
let Koa = require('koa');
let Router = require('koa-router');
let app = new Koa();
let router = new Router();
router.get('/read', (ctx, next) => {
    //有name读name
    let name = ctx.cookies.get("name") || '没有name';
    let age = ctx.cookies.get("age") || '没有age';
    ctx.body = `${name}-${age}`;
});
router.get('/write', (ctx, next) => {
    ctx.cookies.set('name', 'cgp',{
        domain:'127.0.0.1', //写入cookie所在的域名
        path:'/write',    // 写入cookie最大的路径
        maxAge:10*1000,    //Cookie最大有效时长
        httpOnly:false,  // 是否只用于http请求中获取
        overwrite:false  // 是否允许重写
    });
    ctx.cookies.set('age', '9');
    ctx.body = 'write Ok';
});
app.use(router.routes());
app.listen(4000);
```
###### Cookie选项
- domain：写入cookie所在的域名
- path：写入cookie所在的路径
- maxAge：Cookie最大有效时长
- expires：cookie失效时间
- httpOnly:是否只用http请求中获得
- overwirte：是否允许重写

> 看下运行结果吧

![](https://user-gold-cdn.xitu.io/2018/7/31/164ee717c2064d72?w=960&h=394&f=png&s=11403)

#### 未完待续

> [对常用中间件源码感兴趣可以参考下我总结](https://github.com/chaiguanpeng/koa-analysis)

> [更多阅读原文可以看这里](https://chaiguanpeng.github.io/)

> 都看到这里啦，喜欢就点个赞吧
