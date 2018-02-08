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

- [package](#package)

- [flow-类型验证插件](#类型验证插件)

- [react-import](#react-import)

- [worker-js web worker 定义](#worker-js)

- [js-flow-类型定义](#类型定义)

- [高阶组件](#高阶组件)

- [ImageWorker-Component](#ImageWorker-Component)

- [重点🧡ImageWorker-react组件周期](#ImageWorker-react组件周期)

- [重点🌟web-worker-信息传递](#web-worker-信息传递)

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

本次解释的例子

``` js
<ImageWorker src='http://image-url' />
```

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

代码 19-30 

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

### ImageWorker-Component

继承react组件-的主组件

代码 37

``` js
class ImageWorker extends Component<ImageWorkerProps, ImageWorkerState> {

// 用 es6 语法 继承 react.Component
// > 记得用语法转为通用js,比如 babel
// 并加以类型约束
```

#### ImageWorker-react组件周期

代码 38-46

``` js
// 定义局部变量和类型
  image: HTMLImageElement;
  worker = new Worker(URL.createObjectURL(
    new Blob([webWorkerScript], { type: 'application/javascript' })
  )) // 启动以-webWorkerScript-的 Web worker

  state = {
    isLoading: true,
    imgSrc: ''
  }
  // 局部存储
```

在 **ImageWorker Class** 中

第一次运行顺序

`constructor` -> `render` -> `componentDidMount` 

> 分别是 初始化 -> 载入 -> 已载入后

如果 `ImageWorker` 不再挂载

 -> `componentWillUnmount`

> componentWillUnmount == 去除组件之前

---

4. componentWillUnmount 先搞定这里

代码 58-64

去除局部变量 去掉web worker

``` js
  componentWillUnmount() {
    if (this.image) {
      this.image.onload = null;
      this.image.onerror = null;
    }
    this.worker.terminate();
  }
```
---

1. constructor

代码 47-52

``` js
  constructor(props: ImageWorkerProps) {
    super(props); // 父类初始化
    // this.worker == 上面定义的worker
    // 因为初始实例化 ImageWorker
    this.worker.onmessage = (event: Object) => {
      this.loadImage(event.data);
    };
    // 从 webWorkerScript 返回信息 - 获取信息后 - 处理
  }
```

在这里, 就需要注意一下 `props`, 默认 react

``` js
<ImageWorker src='http://image-url' /> // jsx
// jsx -> js
ImageWorker({src:'http://image-url'}) // js
// props == {src:'http://image-url'}

// constructor(props: ImageWorkerProps) 
```

> 获得的用户定义变量-放入-> props -变量对象中

---


2. render

代码 92-103

``` js
  render() {
    const { style, imageClass, containerClass } = this.props;
    return (
      // jsx 用 HTML格式写 函数运行 的语法糖
      <div className={containerClass}>
        {
          // 三元写法在，react的表达中很普遍
          // something? true : false

          // isLoading == true
          this.state.isLoading ? this.renderPlaceholder() :
            <img src={this.state.imgSrc}
              style={{ ...style }} className={imageClass} alt='worker' />
        }
      </div>
    );
  }
```

- this.renderPlaceholder

``` js
  renderPlaceholder() {
    const { placeholder, style } = this.props; 
    // 没有定义, 本次例子中
    if (typeof placeholder === 'function') { 
      const PlaceholderComponent = wrappedComponent(placeholder);
      return <PlaceholderComponent />;
    } else if (typeof placeholder === 'string') {
      return <img src={placeholder} style={{ ...style }} alt='placeholder' />;
    } else {
      return null; // <---- 到这里，接下来->componentDidMount
    }
  }
```

---

3. componentDidMount

``` js
  componentDidMount() {
    this.worker.postMessage(this.props.src); // 传递图片路径 
  }
```

---

回顾一下 web worker 信息的传递顺序-第一步

> 1. `this.worker.postMessage` -> 

> 2. `webWorkerScript` 里面的 监听 `message`函数变量 e.data ->

> 3. `webWorkerScript` 里面的 监听 message函数 `postMessage`(url) ->

> 4. `this.worker.onmessage`

### web-worker-信息传递


1. 上面说了

2. webWorkerScript

``` js
// worker-js-函数
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

- [self.addEventListener worker的API]()

监听获取信息的-函数

将-`图片路径url`- 用 - `fetch` - 获取 - `blob 化` - `返回postMessage`-> `this.worker.onmessage`

3. postMessage 上面说了

4. this.worker.onmessage

``` js
    this.worker.onmessage = (event: Object) => {
      this.loadImage(event.data);
    };
  }
```

- this.loadImage

代码 78-83

``` js
  loadImage = (url: string) => {
    const image = new Image();
    this.image = image;
    image.onload = this.onLoad; // 加载后
    image.src = url;
  }
```

- this.onLoad

代码 85-90

``` js
  onLoad = () => {
    this.setState({
      imgSrc: this.image.src,
      isLoading: false
    }); 
  }
```

`this.setState 改变了局部变量` 会推动 [render 重新运行](#render)