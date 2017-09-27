---
title: Vue2.0内部指令（中）
date: 2017-09-10 10:26:55
tags: [js,vue]
---
## 第四节: v-text & v-html

  在html中输出data值时, 使用 \{ { message }} 。但是这种情况是有弊端的，就是当我们网速很慢或者javascript出错时,会暴露我们的 \{ { message }} 。Vue给我们提供的 v-text就是解决这个问题的。

  ```
  <p>{{ message }}</p>=<p v-text="message"></p>

  ```
  如果在javascript中写有html标签,用v-text是输出不出来的,这时我们就需要用 `v-html`标签
  实例:
  ```
  <div id="app">
      <span v-text="message"></span>
      <span v-html="myHtml"></span>
  </div>
  <script>
      var app=new Vue({
          el:"#app",
          data:{
              message:"hello world",
              myHtml:"<h2>hello world</h2>"
          }
      })
  </script>

  ```
## 第五节: v-on事件的绑定
  可以用v-on指令监听DOM事件来触发一些js代码
  实例:
  ```
  <div id="app">
       本场比赛得分： {{count}}<br/>
       <button v-on:click="add">加分</button>
       <button @click="reduce">减分</button>
</div>
    <script type="text/javascript">
        var app=new Vue({
            el:'#app',
            data:{
                count:1
            },
            methods:{
                add:function(){
                    this.count++;
                },
                reduce:function(){
                    this.count--;
                }
            }
        })
    </script>

  ```
v-on的简单写法，就是用@代替
```
<button @click="reduce">减分</button>
```
## 第六节: v-model指令
  v-model指令即绑定数据源。就是把数据绑定在特定的表单元素上,可以很容易的实现双向数据绑定。

#### 一、文本
```
<div id="app">
    <p>原始数据: {{message}}</p>
    <input type="text" v-model="message">
</div>
<script>
   var app=new Vue({
       el:"#app",
       data:{
           message:"hello world"
       }
   })
</script>
```
#### 二、多行文本
```
<div id="app">
    <p>Message is: {{message}}</p>
   <textarea name="" id="" cols="30" rows="10" v-model="message"></textarea>
</div>
<script>
   var app=new Vue({
       el:"#app",
       data:{
           message:"hello world"
       }
   })
</script>
```
#### 三、复选框
1、单个勾选框
```
<div id="app">
  <input type="checkbox" id="checkbox" v-model="checked">
  <label for="checkbox">{{ checked }}</label>
</div>
<script>
   var app=new Vue({
       el:"#app",
       data:{
           checked:false
       }
   })
</script>
```
2、多个勾选框，绑定到同一个数数组:
```
<div id="app">
    <input type="checkbox" id="checkboxA" v-model="checked" value="A">
    <label for="checkboxA">A</label>
    <input type="checkbox" id="checkboxB" v-model="checked" value="B">
    <label for="checkboxB">B</label>
    <input type="checkbox" id="checkboxC" v-model="checked" value="C">
    <label for="checkboxC">C</label>
    <p>{{checked}}</p>
</div>
<script>
    var app=new Vue({
        el:"#app",
        data:{
            checked:[]
        }
    })
</script>
```
3、单选按钮

```
<div id="app">
    <p>
       <input type="radio" id="one" value="男" v-model="sex">
       <label for="one">男</label>
       <input type="radio" id="two" value="女" v-model="sex">
       <label for="two">女</label>
       <p>您选择的性别是: {{sex}}</p>
   </p>
</div>
<script>
    var app=new Vue({
        el:"#app",
        data:{
            message:"hello world",
            isTrue:true,
            web_names:[],
            sex:'男'
        }
    })
</script>

```
