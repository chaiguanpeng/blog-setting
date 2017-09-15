---
title: Vue2.0 内部指令(上)
date: 2017-09-11 17:13:21
tags: [js,vue]
---
## 第一节: 走进Vue2.0

#### 一、下载Vue2.0的两个版本
  官方网站 ：https://cn.vuejs.org/v2/guide/
  1. 开发版本 : 包含完整的警告和调试模式
  2. 生产版本 : 删除了警告,进行了压缩

#### 二、live-server使用
  用npm进行全局安装(默认你已经会用git)
```
npm install live-server -g
```
在项目目录中打开
```
live-server
```
#### 三、编写一个 hello world代码
```
<div id="app">
       {{message}}
</div>
<script type="text/javascript">
       var app=new Vue({
           el:'#app',
           data:{
               message:'hello Vue!'
           }
       })
</script>
```

## 第二节: v-if v-else v-show指令

#### 一、v-if（是vue的一个内部指令，指令用在我们的html中。）
  用来判断是否加载html的DOM，比如我们模拟一个用户登陆状态，在用户登录后显示用户名称。
  ```
  <div id="app">
        <div v-if="isLogin">你好:peter</div>
        <div v-else>请登陆后操作</div>
  </div>
  <script type="text/javascript">
         var app=new Vue({
             el:'#app',
             data:{
                isLogin:false
             }
         })
  </script>
  ```
这里我们在vue的data里定义了isLogin的值，当它为true时,网页就会显示：你好:peter，如果为false时,就显示请登陆操作。

#### 二、v-show
  调整css中display属性，DOM已经加载,只是CSS控制没有显示出来。
  ```
  <div v-show="isLogin">你好: peter</div>

  ```
#### v-if与v-show区别

v-if：   判断是否加载,可以减轻服务器压力，在需要时加载
v-show:  调整css display属性,可以使客户端操作更加流畅

## 第三节: v-for指令 解决模板循环问题


  
