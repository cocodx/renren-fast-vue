##### 开发基础知识
***
let 声明变量
const 声明常量（只读）
***
```javascript
const person={
    name: "jack",
    age:21,
    language: ['java','js','css']
}
const {name,age,language}=person;

let info = `我是${name},今年${age + 10}了`;

function fun(){
    return "这是一个函数";
}
let info1 = `我是${name},今年${age + 10}了，我想说${fun()}`;
console.log(info);
```

##### 箭头函数

##### 关闭eslint
在根目录下，config文件夹，index.js设置
```javascript
useEslint: false
```
***
##### 三级分类笔记
draggable: 表示 是否开启拖拽功能。boolean类型

allow-drag: 表示 判断节点能否被拖拽 如果没有设置是不能拖动的啊，设置为true就好了

allow-drop: 表示 拖拽节点之后能否被放置。boolean类型【三级分类条件，分级】

node-drop: 表示 拖拽成功之后触发的事件。update更新节点


#### Vue知识点

##### props
组件实例的作用域是孤立的。这意味着不能 (也不应该) 在子组件的模板内直接引用父组件的数据。父组件的数据需要通过 prop 才能下发到子组件中。

也就是props是子组件访问父组件数据的唯一接口。
```javascript
props:{
  attrtype:{
       type: Number,
       default:1
     }
},
```

##### 引入PubSub.js，实现父子组件通信
```javascript
npm i pubsub-js

import PubSub from 'pubsub-js'
Vue.use(PubSub)
```

