### 为了项目的长远发展，renren-fast-vue已迁移到 【[renren-ui](https://gitee.com/renrenio/renren-ui)】里面，以后会在renren-ui里面进行更新维护，感谢支持！ :smiley:
<br>

## renren-fast-vue
- renren-fast-vue基于vue、element-ui构建开发，实现[renren-fast](https://gitee.com/renrenio/renren-fast)后台管理前端功能，提供一套更优的前端解决方案
- 前后端分离，通过token进行数据交互，可独立部署
- 主题定制，通过scss变量统一一站式定制
- 动态菜单，通过菜单管理统一管理访问路由
- 数据切换，通过mock配置对接口数据／mock模拟数据进行切换
- 发布时，可动态配置CDN静态资源／切换新旧版本
- 演示地址：[http://demo.open.renren.io/renren-fast](http://demo.open.renren.io/renren-fast) (账号密码：admin/admin)

## 说明文档
项目开发、部署等说明都在[wiki](https://github.com/renrenio/renren-fast-vue/wiki)中。


## 更新日志
每个版本的详细更改都记录在[release notes](https://github.com/renrenio/renren-fast-vue/releases)中。


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




