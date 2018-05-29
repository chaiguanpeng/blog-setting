---
title: js数组中filter、map、reduce、find等方法实现的原理
date: 2018-05-29 10:44:38
tags: js
---
## filter用法和原理实现
- filter 过滤,filter()使用指定的函数测试所有元素，并创建一个包含所有通过测试的元素的新数组。

```
let arr=[2,4,6,8];
let arr1=arr.filter(function(item){
    return item>5
})
console.log(arr1) //[6,8]
```
- 原理的实现

```
Array.prototype.filter1 = function (fn) {
  let newArr=[];
  for(let i=0;i<this.length;i++){
      fn(this[i]) && newArr.push(this[i]);
  }
  return newArr;
};
let arr=[2,4,6,8];
let arr1=arr.filter1(function(item){
    return item>5
})
console.log(arr1) //[6,8]
```
- 看完之后是不是so easy，其它的几个实现大同小异,建议都手写遍
## map用法和原理实现
- map 映射,map()方法返回一个新数组，数组中的元素为原始数组元素调用函数处理的后值。

```
let arr = ['bob', 'grex', 'tom'];
let arr1 = arr.map(function(item) {
    return `<li>${item}</li>`;
});
console.log(arr1); //[ '<li>bob</li>', '<li>grex</li>', '<li>tom</li>' ]

```
- 原理实现

```
Array.prototype.map = function(fn) {
    let newArr = [];
    for (let i = 0; i < this.length; i++) {
        newArr.push(fn(this[i]))
    };
    return newArr;
}

```
## reduce用法和原理
- reduce() 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。

```
var arr=[2,4,6,8];
let result=arr.reduce(function (val,item,index,origin) {
    return val+item
},0);
console.log(result) //20

```
- reduce原理实现

```
Array.prototype.reduce = function (reducer,initVal) {
    for(let i=0;i<this.length;i++){
        initVal =reducer(initVal,this[i],i,this);
    }
    return initVal
};

```

## find用法和原理实现
- find() 方法返回通过测试（函数内判断）的数组的第一个元素的值。

```
let arr = [1,2,3];
let arr1=arr.find(function (item) {
    return item>=2
});
console.log( arr5); //2

```
- find原理实现

```

Array.prototype.find = function(fn) {
    for (let i = 0; i < this.length; i++) {
        if (fn(this[i])) return this[i]
    }
}

```
## some用法和原理实现
- some() 方法会依次执行数组的每个元素：

 > 如果有一个元素满足条件，则表达式返回true , 剩余的元素不会再执行检测。

 > 如果没有满足条件的元素，则返回false。

```
let arr = [2, 4, 6, 8];
let flag = arr.some(function(item) {
    return item > 5
});
console.log(flag); //true

```
- some原理实现

```

Array.prototype.some=function (fn) {
    for(let i=0;i<this.length;i++){
        if(fn(this[i])) {
            return true
        }
    }
    return false
};

```
## every用法和原理实现
- every方法用于检测数组所有元素是否都符合指定条件（通过函数提供）。

```

let arr = [2, 4, 6, 8];
let flag = arr.every(function(item) {
    return item > 5
});
console.log(flag); //false

```

- 原理实现

```

Array.prototype.every=function (fn) {
    for(let i=0;i<this.length;i++){
        if(!fn(this[i])) {
            return false
        }
    }
    return true
};

```
