---
title: js实现数据与视图的双向绑定
date: 2019-12-06 09:14:25
tags: js
categories: 前端
---

## 实现思路 Object.defineProperty

```
 let data = {
    age: 10
  };
  Object.defineProperty(data, 'age', {
    get() {
      console.log('我被获取的了');
      return this.age;
    },
    set(newValue) {
      console.log('我被设置了');
    }
  });
  let a = data.age;
  data.age = 20;

```

我们定义一个 data 对象，为他的 age 属性定义 get，set 方法，此时我们去给 a 赋值时便会调用 get 方法，而为 data.age 赋值时，会调用 set 方法，因此利用这个我们便可以来实现简单的双向绑定了，以 vue 为例。

## 创建 myVue.js 用来创建属于我们自己的 vue 对象

为了将我们创建的对象里面的属性全部注册到 Object.defineProperty 中，我们创建方法 observable

```
function observable(obj) {
  if (!obj || Object.prototype.toString.call(obj) !== '[object object]') return;
  let keys = Object.keys(obj);
  keys.forEach((key) => {
    handleDefine(obj, key);
  });
}

function handleDefine(obj, key) {
  Object.defineProperty(obj, key, {
    get() {
      console.log(`${key}获取到了`);
      return this[key];
    },
    set(value) {
      console.log(`${key}赋值${value}`);
    }
  });
}
```

## 创建 index.html 来渲染我们的视图并观察效果

将 handle.js 引入 index.html 中
在 index.html 定义一个 data 对象，并创建对应的 input 和视图

```
<div id="vue">
      <p>name:<span data-view="name"></span></p>
      <p>age:<span data-view="age"></span></p>
      name:<input data-model="name" /><br />
      age:<input data-model="age" />
</div>


let data = {
    name:'pipixia',
    age:10
}
```

## 为我们需要的 data-model 对象注册 input 事件

为带有 data-model 属性的标签，注册 input，并创建 myVue 类，el 代表我们双向绑定生效的标签块

```
//获取所有的model对象并为它们注册input事件

function handleDomInput(el, data) {
  let nodeList = document.querySelectorAll(`${el} [data-model]`);
  nodeList.forEach((item) => {
    let key = item.dataset.model;
    item.addEventListener('input', function(node) {
      data[key] = node.target.value;
    });
  });
}


//新建自己vue类
class myVue {
  constructor({ el, data }) {
    this.el = el;
    this.data = data;
    //分发双向绑定事件
    observable(this.data);
    //注册input事件
    handleDomInput(this.el, this.data);
  }
}

```

## 将改变的数据渲染到视图中

这一步操作，我们就可以利用 Object.defineProperty 中的 set 事件来操作

### 创建赋值的方法

```

//获取所有的view对象并为他们复职
function handelDomView(key, value, el) {
  let node = document.querySelectorAll(`${el} [data-view = ${key}]`);
  node.forEach((item) => {
    item.innerText = value;
  });
}
```

### 将方法加入 Object.defineProperty 中并将 el 传入 observable 中

```
function handleDefine(obj, key, el) {
  Object.defineProperty(obj, key, {
    get() {
      return this[key];
    },
    set(value) {
      handelDomView(key, value, el);
    }
  });
}

//获取所有的view对象并为他们复职
function handelDomView(key, value, el) {
  let node = document.querySelectorAll(`${el} [data-view = ${key}]`);
  node.forEach((item) => {
    item.innerText = value;
  });
}

//新建自己vue类
class myVue {
  constructor({ el, data }) {
    this.el = el;
    this.data = data;
    //分发双向绑定事件，并传入区域块 el
    observable(this.data, el);
    handleDomInput(this.el, this.data);
  }
}


```

## 数据初始化加入 initData

在 myVue 中将 this.data 拷贝一份

```
//新建自己vue类
class myVue {
  constructor({ el, data }) {
    this.el = el;
    this.data = data;
    //拷贝
    let initData = JSON.parse(JSON.stringify(this.data));
    //分发双向绑定事件
    observable(this.data, el, initData);
    handleDomInput(this.el, this.data, initData);
  }
}
```

并且在注册双向绑定和分发 input 事件时，进行初始化

```
function observable(obj, el, initData) {
  if (!obj || Object.prototype.toString.call(obj) !== '[object Object]') return;
  let keys = Object.keys(obj);
  keys.forEach((key) => {
    //初始化
    handelDomView(key, initData[key], el);
    handleDefine(obj, key, el);
  });
}


function handleDomInput(el, data, initData) {
  let nodeList = document.querySelectorAll(`${el} [data-model]`);
  nodeList.forEach((item) => {
    let key = item.dataset.model;
    //初始化
    item.value = initData[key];
    item.addEventListener('input', function(node) {
      data[key] = node.target.value;
    });
  });
}

```

### 全部代码

```
index.html

<div id="vue">
    <p>name:<span data-view="name"></span></p>
    <p>age:<span data-view="age"></span></p>
    name:<input data-model="name" /><br />
    age:<input data-model="age" />
</div>


new myVue({
    el: '#vue',
    data: {
      name: 'pipixia',
      age: 10
    }
  });

myVue.js

function observable(obj, el, initData) {
  if (!obj || Object.prototype.toString.call(obj) !== '[object Object]') return;
  let keys = Object.keys(obj);
  keys.forEach((key) => {
    handelDomView(key, initData[key], el);
    handleDefine(obj, key, el);
  });
}

function handleDefine(obj, key, el) {
  Object.defineProperty(obj, key, {
    get() {
      return this[key];
    },
    set(value) {
      handelDomView(key, value, el);
    }
  });
}

//获取所有的model对象并为它们注册input事件

function handleDomInput(el, data, initData) {
  let nodeList = document.querySelectorAll(`${el} [data-model]`);
  nodeList.forEach((item) => {
    let key = item.dataset.model;
    item.value = initData[key];
    item.addEventListener('input', function(node) {
      data[key] = node.target.value;
    });
  });
}

//获取所有的view对象并为他们复职
function handelDomView(key, value, el) {
  let node = document.querySelectorAll(`${el} [data-view = ${key}]`);
  node.forEach((item) => {
    item.innerText = value;
  });
}

//新建自己vue类
class myVue {
  constructor({ el, data }) {
    this.el = el;
    this.data = data;
    //分发双向绑定事件
    let initData = JSON.parse(JSON.stringify(this.data));
    observable(this.data, el, initData);
    handleDomInput(this.el, this.data, initData);
  }
}
```
