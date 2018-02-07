# react-worker-image

「 React组件通过web worker获取图像资源 」

[![explain](http://llever.com/explain.svg)](https://github.com/chinanf-boy/Source-Explain)

Explanation

> "version": "1.0.0"

[github source](https://github.com/nitish24p/react-worker-image)

~~[english](./README.en.md)~~

---

react-worker-image是一个用于通过web worker加载图像的React组件。从而不会阻塞主线程并加快页面加载时间。

---

## 示例展示

在两种情况下, 可以留意加载时间

1. 第一个是通过webworker

2. 第二个是<img 

![demo 展示](./screen.gif)

---

# 目录

---

## package

``` js
"main": "index.js",
```

主文件 ·index.js·

但是源代码中看不到·index.js· 再看

``` js
  "scripts": {
    "build": "NODE_ENV=production babel lib/ImageWorker.js --out-file index.js", // 原来在这里, 看来真正的文件是
    //lib/ImageWorker.js
    "prepublish": "npm run build",
    "eslint": "node_modules/.bin/eslint -c ./.eslintrc.json ./lib/*"
  },
```
## ImageWorker


### 类型验证插件

代码 1

```
/* @flow */
```

### react-import

代码 3

``` js
import React, { Component } from 'react'; // 导入 react

```

### worker-js

代码 5-16

``` js
// worker-函数
const webWorkerScript = ` 
  self.addEventListener('message', event => {
    const url = event.data;
    fetch(url, {
        method: 'GET',
        mode: 'no-cors',
        cache: 'default'
    }).then(response => {
        return response.blob();
    }).then(_ => postMessage(url));
  })
`;
```

### 类型定义

代码 19-30 ``

``` js
type ImageWorkerProps = {
  src: string,
  placeholder?: string | Function,
  style?: Object,
  imageClass?: string,
  containerClass?: string,
}

type ImageWorkerState = {
  isLoading: boolean,
  imgSrc: string
}
```

### 高阶组件

高阶组件是，输入一个组件`input`, `return`一个带有-`input`组件变量的-> `output`组件

代码 33-35

``` js
const wrappedComponent = WrappedComponent => props => {
  return <WrappedComponent {...props} />;
};
```

### ImageWorker

继承react组件-的主组件

代码 37

``` js
class ImageWorker extends Component<ImageWorkerProps, ImageWorkerState> {

// 用 es6 语法 继承 react.Component
// > 记得用语法转为通用js,比如 babel
// 并加以类型约束
```


代码 38-46

``` js
// 定义局部变量和类型
  image: HTMLImageElement;
  worker = new Worker(URL.createObjectURL(
    new Blob([webWorkerScript], { type: 'application/javascript' })
  ))

  state = {
    isLoading: true,
    imgSrc: ''
  }
```