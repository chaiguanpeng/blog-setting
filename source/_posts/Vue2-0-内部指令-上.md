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
  用npm进行全局安装
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
