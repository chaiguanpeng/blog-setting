---
title: Vue2.0内部指令(下)
date: 2017-09-9 11:41:53
tags: [js,vue]
---
## 第七节:v-bind指令
  v-bind是处理HTML中的标签属性的，例如我们绑定 `<img>`上的src进行动态赋值。
  实例:
  ```
<div id="app">
    <img v-bind:src="imgSrc"  width="200px">
</div>
<script>
var app=new Vue({
    el:'#app',
    data:{
          imgSrc:'http://baidu.com/wp-content/uploads/2017/02/vue01-2.jpg'
     }
})
</script>

  ```
v-bind缩写
```
<a v-bind:href="url"></a>
<!-- 缩写 -->
<a :href="url"></a>
```

绑定CSS样式：在绑定CSS样式时,绑定的值必须在vue中的data属性中进行声明。

#### 1、直接绑定class样式
  ```
  <div id="app">
      <div :class="className">1、绑定classA</div>
  </div>
  <script>
  var app=new Vue({
      el:'#app',
      data:{
            className:"classA"
       }
  })
  </script>

  ```
#### 2、绑定classA并进行判断，在isOk为true时显示样式，否则不显示
```
<div id="app">
    <div :class="{className : isOk}"> 2、绑定classA并进行判断</div>
</div>
<script>
var app=new Vue({
    el:'#app',
    data:{
        isOk:true
     }
})
</script>

```

#### 3、绑定class中的数组
```
<div id="app">
    <div :class="[class1,class2]">3、绑定class中的数组</div>
</div>
<script>
    var app=new Vue({
        el:'#app',
        data:{
            class1:"classA",
            class2:"classB"
        }
    })
</script>

```
#### 4、绑定class中使用三元表达式判断

```
<div id="app">
    <div :class="isOk?classA:classB">4 绑定class中的三元运算符</div>
</div>
 <script>
     var app=new Vue({
         el:'#app',
         data:{
              isOk:true,
             classA:"classA",
             classB:"classB"
         }
     })
 </script>

```
#### 5、绑定style
```
<div id="app">
     <div :style="{color:red,fontSize:font}">5 绑定style</div>
</div>
 <script>
     var app=new Vue({
         el:'#app',
         data:{
           red:"red",
           font:"16px",
         }
     })
 </script>

```
#### 6、用对象绑定style样式
```
<div id="app">
     <div :style="styleObj">6、用对象绑定style样式</div>
</div>
 <script>
     var app=new Vue({
         el:'#app',
         data:{
           styleObj:{
                 color:'green',
                 fontSize:'24px'
             }
         }
     })
 </script>

```

## 第八节:其他背部指令(v-pre & v-cloak & v-once)

#### 1、v-pre指令
在模板中跳过vue的编译，直接输出原始值。就是在标签中加入v-pre就不会输出vue中的data值了。
```
<div v-pre>{{message}}</div>

```
这时并不会输出我们的message值,而是直接在网页中显示 `{{message}}`

#### 2、v-cloak指令
在vue渲染完指定的DOM后才进行显示。它必须和CSS样式一起使用

```
[v-cloak]{
  display:none;
}
<div v-pre>{{message}}</div>

```
#### 3、v-once指令
在第一次DOM时进行渲染,渲染完成后视为静态内容,跳出以后的渲染过程
```
<div>{{message}}</div>
<div><input type="text" v-model="message"></div>
<div v-once>{{message}}</div>

```
