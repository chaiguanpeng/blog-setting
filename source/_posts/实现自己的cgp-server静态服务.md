---
title: 实现自己的cgp-server静态服务
date: 2018-07-24 15:26:22
cdn: 'header-on'
header-img: "/images/poem1.jpg"
tags: node
---
#### 序言
> 手写一个静态服务可以对node中http模块有更深的理解，这是我们的初衷。http-server相信大家都用过，这里我们要实现类似个功能。功能如下
- 启动我们写好的模块后，输入localhost:3000打开我们public目录下的文件(默认打开index.html)
- 用到debug插件，主要用于在命令行输出一些日志，我们只用基本的功能，所以没有难点。[不会戳这里](https://www.npmjs.com/package/debug)
- 可能用到chalk插件，就是把命令行输出的日志五颜六色，变得好看，没什么太大作用，[用法戳这里](https://www.npmjs.com/package/chalk)

#### 准备工作
##### 我们的目录解构如下


![](https://user-gold-cdn.xitu.io/2018/7/23/164c777502e521e2?w=540&h=426&f=png&s=6185)

- 大家应该一看就懂啦，启动我们服务，自动打开public/index.html
- bin/www.js 是我们后面用命令行启动服务的配置
- public是我们的静态目录
- app.js  主文件
- config.js 配置文件
- tmpl.html是我们用ejs编译的模板，后面讲到

#### 先写最简单的config.js

```
let path = require('path');
let config = {
    hostname:'127.0.0.1', //默认主机
    port:3000,  //默认端口
    dir:path.join(__dirname,'../public') //默认打开的目录(绝对路径)
};
module.exports = config;
```
> 以上代码都能看得懂，下面开始写我们主文件

#### 核心代码 app.js

##### 1、引入所需的依赖包

```
let http = require('http');
let url = require('url');
let path = require('path');
let util = require('util');
let fs = require('fs');
let zlib = require('zlib');
let mime = require('mime'); // 得到内容类型
let debug = require('debug')('*'); // 打印输出 会根据环境变量控制输出
let chalk = require('chalk'); // 粉笔
let ejs = require('ejs'); // 模板引擎

//先声明好，下面解释
let config = require('./config');
let stat = util.promisify(fs.stat);//promise化 fs.stat方法
let readdir = util.promisify(fs.readdir);
let template = fs.readFileSync(path.join(__dirname,'tmpl.html'),'utf8'); //读取ejs的模板文件
```
- mime解析文件给你内容类型，[用法](https://www.npmjs.com/package/mime)
- ejs渲染引擎，我们用最简单功能，不会也能看懂

##### 2、http模块开启服务

```
/*运行的条件 指定主机名
* 指定启动的端口号
* 指定运行的目录
 */
let config = require('./config'); //引入配置文件
class Server { //声明类
    constructor() {
        this.config = config; //讲配置挂载再我们的实例上
    }
    handleRequest(req,res){ //确保这里的this都是实例

    }
    start(){//服务开始的方法
        let server =http.createServer(this.handleRequest.bind(this));
        let {hostname,port} = this.config; //解构主机名和端口
        server.listen(port,hostname);
        debug(`http://${hostname}:${port} start`) //命令行中打印
    }
}
//开启一个服务
let server = new Server();
server.start(); //调用start方法

```
> 截至到目前位置，简单的服务已经开启了，先来测试下效果吧

![](https://user-gold-cdn.xitu.io/2018/7/23/164c5db515379d7a?w=1161&h=525&f=png&s=26003)

> 完美，控制台打印出了内容，

##### 3、实现handleRequest方法，即处理请求逻辑

> 列出我们要做什么
- 解析url的路径名
- 与默认配置中路径(G://cgp-server/public)拼接
- 判断是文件还是文件夹还是404

```
let stat = util.promisify(fs.stat);//promise化 fs.stat方法
 async  handleRequest(req,res){ //确保这里的this都是实例
        let {pathname} = url.parse(req.url,true); //获取url的路径
        let p = path.join(this.config.dir,pathname); // 可能是G:/cgp-server/public 可能是G://cgp-server/public/index.html
        //1、根据路径 返回不同结果 如果是文件夹 显示文件夹里的内容
        //2、如果是文件 显示文件的内容
        try{
            let statObj=await stat(p);
        }catch (e) {
            //文件不存在情况
            this.sendError(req,res,e)
        }
    }
```
> try catch用于捕获错误，当文件不存在，调用sendError方法，先来实现这个错误的处理方法

##### 4、文件不存在的逻辑，sendError()

```
sendError(req,res,e){
        debug(util.inspect(e)); //输出错误,util模块提供方法
        res.statusCode = 404;
        res.end('Not Found');
    }
```
> 写了这么多了，测试下错误文件能否打印错误
![](https://user-gold-cdn.xitu.io/2018/7/23/164c60af4ed79d64?w=1164&h=711&f=png&s=36083)

> 测试完美,此时我们应该判断打开的是文件还是目录，并给对应的方法，下面我们开始目录的渲染方法

#### 5、ejs渲染目录列表
- 先声明一个template模板，挂载到实例上
```
let template = fs.readFileSync(path.join(__dirname,'tmpl.html'),'utf8'); //读取ejs的模板文件
class Server{
    constructor(){
        this.template = template //挂载到实例上
    }
}
```
- 如果是目录，渲染出一个html页展示目录结构

```
if(statObj.isDirectory()){
                //如果是目录 列出目录内容可以点击
                let dirs = await readdir(p); //public下面的目录结构=>[index.html,style.css]
                dirs =dirs.map(dir=>{
                    return {
                        filename:dir,
                        path:path.join(pathname,dir)
                    }
                });
                //dirs就是要渲染的数据
                //格式如下[{filename:index.html,path:'/index.html'},{{filename:style.css,path:''/style.css}}]
                let str =ejs.render(this.template,{dirs}); //ejs渲染方法
                // console.log(str);
                res.setHeader('Content-Type', 'text/html;charset=utf-8');
                res.end(str);
            }
```
- 我们来看下tmpl.html模板是怎么写的,[ejs用法](https://ejs.bootcss.com/),我们只用最简单的，所以应该能看懂

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
//循环dirs中的内容到页面中
<% dirs.map(item=>{%>
    <li><a href="<%=item.path%>"><%=item.filename%></a></li>
<%})%>
</body>
</html>

```
> 渲染目录结构，我们已经写完了，测试下看能不能运行

![](https://user-gold-cdn.xitu.io/2018/7/23/164c79724979531b?w=1334&h=650&f=jpeg&s=80623)
> 目前来看，无bug，接下来实现如果是文件的话，直接把文件内容渲染出来

#### 6、文件的渲染方法,即this.sendFile()方法
```
sendFile(req,res,p,statObj){
     res.setHeader('Content-Type', mime.getType(p) + ';charset=utf-8');
        fs.createReadStream(p).pipe(res);//可读流pipe到可写流
}
```
> 功能已经实现拉，不信我们测下

![](https://user-gold-cdn.xitu.io/2018/7/23/164c79fb1a9bbf5d?w=1215&h=637&f=png&s=34593)

- 功能已经实现，但我们的要求在增加三个功能
- 1、检测是否支持缓存
- 2、检测是否支持压缩
- 3、检测是否支持范围请求

##### 6.1增加缓存功能
- 修改下sendFile()方法添加三个功能

```
 sendFile(req,res,p,statObj){
        // 1、检测是否有缓存
        if(this.cache(req,res,p,statObj)){ //如果有缓存
            res.statusCode = 304;
            res.end();
            return
        }
        //2、检测是否支持压缩
            ....
        //3、检测是否有范围请求
            ....

    }
```
- cache()缓存方法
> 缓存有两种方式，强制缓存和协商缓存
- 强制缓存 服务端Catch-Control 、 Expires
- 协商缓存 服务端Last-Modified 、Etag
- 协商缓存 客户端if-modified-since  if-none-match 与服务端对应
- 贴下百度缓存的解构给大家看下
![](https://user-gold-cdn.xitu.io/2018/7/24/164ca173365aef54?w=829&h=522&f=png&s=24716)
> 看完这个图，相信大家应该懂啦。下面开始写缓存方法

```
cache(req,res,p,statObj){  //实现缓存
        /* 强制缓存 服务端 Cache-Control Expires
            协商缓存  服务端 Last-Modified Etag
            协商缓存  客户端 if-modified-since  if-none-match
            etag ctime + 文件的大小
            Last-modified ctime
            强制缓存
            */
        res.setHeader('Cache-Control', 'no-cache');
        res.setHeader('Expires', new Date(Date.now() + 10 * 1000).toGMTString());//10秒后重新发请求
        let etag = statObj.ctime.toGMTString() + statObj.size; //文件修改时间和文件大小
        let lastModified = statObj.ctime.toGMTString(); //文件的修改时间
        res.setHeader('Etag', etag);
        res.setHeader('Last-Modified', lastModified);
        let ifNoneMatch = req.headers['if-none-match'];
        let ifModifiedSince = req.headers['if-modified-since'];
        if (etag != ifNoneMatch) { //不相等，不走缓存
            return false;
        }
        if (lastModified != ifModifiedSince) { //同理
            return false;
        }
        return true; //否则走缓存
    }

```
> 缓存功能写完了，我们测试下设置的头有没有添加上

![](https://user-gold-cdn.xitu.io/2018/7/24/164ca27a1ea7b284?w=1154&h=881&f=png&s=33747)

> 缓存我们就已经实现啦

##### 6.2 实现压缩功能
- node中zlib提供压缩功能，这里就不讲怎么用啦，[用法戳官网](http://nodejs.cn/api/zlib.html)

```
gzip(req,res,p,statObj){
        // 客户端 Accept-Encoding: gzip, deflate, br
        // 服务端 Content-Encoding: gzip
        let encoding = req.headers['accept-encoding']; //获取请求头的接收的压缩格式
        if (encoding) {
            if (encoding.match(/\bgzip\b/)) {
                res.setHeader('Content-Encoding', 'gzip')
                return zlib.createGzip();//返回一个gzip的压缩流
            } else if (encoding.match(/\bdeflate\b/)) {
                res.setHeader('content-encoding', 'deflate');
                return zlib.createDeflate(); //返回createDeflate的压缩流
            } else {
                return false; //否则不支持压缩
            }
        } else {
            return false;//否则不支持压缩
        }
    }
```
- 修改下sendFile()方法
```
sendFile(req,res,p,statObj){
        // 1、检测是否有缓存
        if(this.cache(req,res,p,statObj)){ //如果有缓存
            res.statusCode = 304;
            res.end();
            return
        }
        //2、检测是否支持压缩
        res.setHeader("Content-Type",mime.getType(p)+";charset=utf8");
        let compress =this.gzip(req,res,p,statObj);
        if(compress){ //检测是否压缩。返回的是压缩流
           return fs.createReadStream(p).pipe(compress).pipe(res);
        }else{ //不支持压缩直接把文件读出来即可
           return fs.createReadStream(p).pipe(res)
        }
        //3、检测是否有范围请求
            ....

    }
```
> 用1.txt文件测试下

![](https://user-gold-cdn.xitu.io/2018/7/24/164ca32aec8db478?w=570&h=776&f=png&s=26553)

> 目前来看都还ok,还剩最后一个功能，实现范围请求

##### 6.3 实现范围请求功能
- 客户端发送Range:bytes=0-3
- 服务端对应Accept-Range:bytes Content-Range:bytes 0-3/xxx Content-Length:xxx
> 由于可能同时会有压缩和范围请求，我们稍微改下前面的代码

```
sendFile(req,res,p,statObj){
      // 1、检测是否有缓存
          ....
      //2、检测是否支持压缩同时加上范围请求
      res.setHeader("Content-Type",mime.getType(p)+";charset=utf8");
      let compress =this.gzip(req,res,p,statObj);
      let {start,end} = this.range(req,res,p,statObj); //解构开始和结束的位置
      if(compress){ //检测是否压缩。返回的是压缩流
         return fs.createReadStream(p,{start,end}).pipe(compress).pipe(res);
      }else{
          // res.setHeader("Content-Type",mime.getType(p)+";charset=utf8");
         return fs.createReadStream(p,{start,end}).pipe(res)
      }

  }

```
- range()范围请求的方法

```
range(req, res, statObj, p) {
        //客户端 Range:bytes=0-3
        //服务端 Accept-Range:bytes Content-Range:bytes 0-3/8777

        let range = req.headers['range']; //如果有范围请求
        if (range) {
            let [, start, end] = range.match(/(\d*)-(\d*)/); //解构出开始和结束的位置
            start = start ? Number(start) : 0; //start设置默认值
            end = end ? Number(end) : statObj.size - 1; //end设置默认值
            res.statusCode = 206; //状态码 206范围请求
            res.setHeader('Accept-Ranges',"bytes");
            res.setHeader('Content-Length',end-start+1);
            res.setHeader('Content-Range',`bytes ${start}-${end}/${statObj.size}`);
            return {start,end};
        }else {
            return {start:0, end:statObj.size};
        }
    }

```
> 基本功能已经实现，测试下代码,我们用curl工具发送请求，[用法](https://www.cnblogs.com/guixiaoming/p/8507268.html)

- 1.txt的内容123456789。我们只想要前4个字符
![](https://user-gold-cdn.xitu.io/2018/7/24/164caf0d3a7bd2bd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
> 测试完美，接下来我们还想实现输入cgp-server ，自动开启浏览器，打开目录。我们需要引用一个模块 yargs

#### 7、yargs模块配置命令行的输入

- [yargs配置用法](https://www.npmjs.com/package/yargs)

- 这里我们只用最基本用法，一看就懂，详细了解请看官网
##### 7.1 npm link

- npm link命令可以将一个任意位置的npm包链接到全局执行环境，从而在任意位置使用命令行都可以直接运行该npm包。

![](https://user-gold-cdn.xitu.io/2018/7/24/164caf9a82ad5983?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 7.2 修改下package.json文件

![](https://user-gold-cdn.xitu.io/2018/7/24/164cafb8c31ae405?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##### 7.3 www.js的配置

###### 7.3.1我们把app.js主文件导出给www.js使用

![](https://user-gold-cdn.xitu.io/2018/7/24/164cafd7eb878ee0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

###### 7.3.2修改www.js文件

```
#! /usr/bin/env node   //执行命令后会执行 bin/www.js
const yargs = require('yargs');
let argv = yargs.option('port',{ //yargs的基础用法
    alias: 'p', //别名
    default: 3000, //默认值
    description:'this is port',  //描述
    demand:false // 是否必须
}).option('hostname',{
    alias: 'h',
    default: 'localhost',
    description:'this is hostname',
    demand:false
}).option('dir',{
    alias: 'd',
    default: process.cwd(),
    description:'this is cwd',
    demand:false
}).usage('cgp-server  [options]' ).argv;

//开启服务
let Server = require('../src/app.js');
new Server(argv).start();

// 判断是win还是mac平台
let platform = require('os').platform();
//开启子进程
let {exec} = require('child_process');
//win系统   win32
if(platform==="win32"){
    exec(`start http://${argv.hostname}:${argv.port}`)
}else {
    exec(`open http://${argv.hostname}:${argv.port}`)
}

```
- 简单介绍yargs用法。然后我们输入 cgp-server --help看效果

```
yargs.option('port',{ //yargs的基础用法
    alias: 'p', //别名
    default: 3000, //默认值
    description:'this is port',  //描述
    demand:false // 是否必须
})

```

![](https://user-gold-cdn.xitu.io/2018/7/24/164cb06dcddbb3ad?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 解释下流程，先开启服务，然后判断系统，再然后根据不同的平台执行自动打开浏览器

> 测试下看能不能启动

![](https://user-gold-cdn.xitu.io/2018/7/24/164cb0a6e27564c1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 结尾

> 如果你能看到这里，真的不容易，点个赞再走吧，[源码分享给你](https://github.com/chaiguanpeng/cgp-server)
