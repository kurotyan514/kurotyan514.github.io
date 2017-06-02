---
layout: single
title: "[ES6]Object.assign"
categories:
  - ES6
tags:
  - ES6
---
# Object.assign是用來？
## 最簡單的用法應該就是拿來 merge Hash 用的
```js
let person = { name: 'sean' };
let age = { age: 18 };
let man = Object.assign({}, person, age);
console.log(man); // { name: 'sean', age: 18 }
```
## 在給參數帶預設值的時候也會用到，
```js
//以前可能會有的實作
function profile (options = {}){
  let name = option.name || 'sean'
  let age = option.age || 18
   //.....很多參數
  return {name, age};
}
//用Object.assign帶預設值
function profile (options = {}){
  let defaults = {
    name: 'sean',
    age: 18
  }
  let result = Object.assign({}, defaults, options)
  return result;
}

profile({name: 'sean'})

```
# Object.assign 的運作
Object.assign() 可以帶多個參數， 越後面的參數會 merge 前面的參數，但不會改變原本變數的值，會被改變的只有第一個參數而已。
```js
let option1 = { name: 'john'};
let option2 = { sex: 'female' };
let option3 = { age: 18, sex: 'male' };
let option4 = { name: 'sean', age: 20};
let original = {name: 'xxx', age: 1, sex: 'xxx'}
let result = Object.assign(original, option1, option2, option3, option4);
console.log(result) //{ name: 'sean', sex: 'male', age: 20 }
console.log(original) //{ name: 'sean', sex: 'male', age: 20 }
console.log(option2) //{ sex: 'female' };
```
# Object.assign 還可以這樣用
```js
var Person = {
  profile() { return (`Hello , I am ${this.name}`) }
};
var json_str = '{ "name":"sean", "age": 18 }';

var man = Object.assign( JSON.parse(json_str) , Person );
console.log( man );  // { name: "sean", age: 18, profile: [Function: profile] }
console.log( man.profile() ); // Hello , I am sean
```
# Object.assign clone 要注意的點
Object.assign 只會針對子項目 clone ，如果子項目還有再下一層的話會建立references
詳細的可以參考 [JavaScript 的 Object.assign 陷阱](https://jigsawye.com/2015/10/06/javascript-object-assign/)

# References
[MDN-Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

[JavaScript 的 Object.assign 陷阱](https://jigsawye.com/2015/10/06/javascript-object-assign/)

[オブジェクトの値をコピーするObject.assign()](http://numb86-tech.hatenablog.com/entry/2016/10/27/123806)

[Object.assign は何をする為にあるんや？](http://takuya-1st.hatenablog.jp/entry/2017/02/23/225731)

[ES2015系列(二) 理解Object.assign](https://cnodejs.org/topic/56c49662db16d3343df34b13)

[ es6 javascript对象方法Object.assign()](http://blog.csdn.net/qq_30100043/article/details/53422657)
