---
title: 初识react(一) 揭开jsx语法和虚拟DOM面纱
date: 2018-09-13 20:23:37
tags: react
---
## 为什么学习react
react以声明式、组件化、一次学习随处编写的理念深受广大开发者热爱，其重要程度不言而喻。因此值得我们深入学习，此专题主要分享些react中部分源码解析。

## jsx简介(js +xml)
- 看一段代码，然后在逐一解释

```
const ele = <h1 className={"box"} style={{fontSize:"20px",color:"red"}}>hello</h1>;
```
> 看不明白没关系，下面我们一步步解释下
- jsx语法中用className代替html中class
- jsx中{}可以存放表达式(有返回值)
- style写法就如上面。放在对象中{color:'red'}
> 这段代码并不是合法的js代码，它是一种被称为jsx的语法扩展，通过它我们就可以很方便的在js代码中书写html片段。
本质上，jsx是语法糖，上面这段代码会被babel转换成如下代码

![](https://user-gold-cdn.xitu.io/2018/7/18/164ac4c48251cdab?w=1506&h=222&f=jpeg&s=25412)
> 分析下，对于React.createElement(xxx)中的参数是如何转变出来的，肯定是用一些正则匹配出来的，我们在这里暂不做重点解释。我们重点关注createElement()方法的实现 和render()方法实现

- createElement()方法。 ->转变为虚拟DOM
- render()方法。        ->讲虚拟DOM转变真实DOM

## 虚拟DOM究竟是啥
> 我们可以把ele打印出来看看虚拟DOM的样子

![](https://user-gold-cdn.xitu.io/2018/7/18/164ac5c3557ce0e4?w=1175&h=612&f=png&s=22488)
- 以上就是最简单虚拟DOM的样子，就是一个对象。
- 其中最主要关注的是的type(类型)、props(属性)我们关注的重点

## createElement()方法的实现

```
let jsxObj = createElement(
    'div',
    null,
    createElement(
        'h1',
        {style: {fontSize: '20px'}, className: 'box'},
        'hello'
    ),
     createElement(
        'h1',
        {style: {fontSize: '20px'}, className: 'box'},
        'world'
    )
);

```
> 假设我们已经有上述的结构，我们来实现createElement()方法转化成虚拟DOM。

```

<div> <h1 style={{fontSize:'20px'}} className='box'>hello</h1></div>
```

>  正则匹配到createElement(xxx)中的各个参数的，这里不做重点介绍正则

- 实现createElement方法

```
function createElement(type, props, ...childrens) {
    return {
        type: type,
        props: {
            ...props,
            children: childrens.length <= 1 ? childrens[0] : childrens
        }
    };
}
```
- 有几点需要注意 {...null}不会报错，会返回{}

![](https://user-gold-cdn.xitu.io/2018/7/18/164ac97429856450?w=246&h=50&f=png&s=617)
- 打印下jsxObj的结果


![](https://user-gold-cdn.xitu.io/2018/7/18/164aca6b24456109?w=1326&h=544&f=png&s=26395)

> 完美，其实createElement()转化为虚拟DOM还是比较简单的，下面来实现讲虚拟DOM转化为真实的DOM呢

## render()方法的实现
> 我们需要这样调用render方法。 render(jsxObj,container) 。jsxObj-> 虚拟的DOM对象 、container->挂载到的节点

```
function render(jsxObj, container) {
    //解构types和props解构props中的children
    let {type, props} = jsxObj, {children} = props;    
    let newElement = document.createElement(type);//创建type类型的DOM元素
    for (let attr in props) { //循环props
        if (!props.hasOwnProperty(attr)) break;
        switch (attr) {
            case 'className': //attr为className,增加class="xxx"属性
                newElement.setAttribute('class', props[attr]);
                break;
            case 'style': //attr为styel，js实现增加样式
                let styleOBJ = props['style'];
                for (let key in styleOBJ) {
                    if (styleOBJ.hasOwnProperty(key)) {
                        newElement['style'][key] = styleOBJ[key];
                    }
                }
                break;
            case 'children':
                let childrenAry = props['children'];
                //childrenAry为数组-> childrenAry,为str的话 ->[str] ,为空的话-> [].统一转变数组便于循环
                childrenAry = childrenAry instanceof Array ? childrenAry : (childrenAry ? [childrenAry] : []);
                childrenAry.forEach(item => {
                    if (typeof item === 'string') {
                        //文本节点，直接增加到元素中
                        newElement.appendChild(document.createTextNode(item));
                    } else {
                        //新的JSX元素，递归调用RENDER，只不过此时的容器是当前新创建的newElement
                        render(item, newElement);
                    }
                });
            default:
                newElement.setAttribute(attr, props[attr]);
        }
    }
    container.appendChild(newElement);
}
```
- 代码注释还算清晰，相信大家看的懂，把我认为不好理解的给大家解释下
- props解构为 {className:'xxx',children:array or string or 空},空代表啥也没有。比如 `<h1></h1>`
- props.hasOwnProperty(attr)可以忽略掉继承过来的属性，即 如果是继承过来的属性，会返回false。为了严谨写上
- childrenAry可能是array or string or 空。我们统一都变成数组，便于后面我们遍历元素
> 写了这么多，来测试下吧

![](https://user-gold-cdn.xitu.io/2018/7/18/164acbf6b68e9247?w=1824&h=441&f=png&s=19600)

> 完美，到目前来看一切正常。附上源码。
[源码直通车](https://github.com/chaiguanpeng/react-code-analysis)
