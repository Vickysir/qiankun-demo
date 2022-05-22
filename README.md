### 目录

[主应用配置](#主应用配置)
[微应用配置](#微应用配置)
[部署](#部署)

### 主应用配置

#### 主应用 main-react

1. 用脚手架创建工程

```
$ yarn create react-app xxx-demo

# or

$ npx create-react-app xxx-demo
```

2. 安装 qiankun

```
$ yarn add qiankun

# or

$ npm i qiankun -S
```

3. 引入乾坤，注册子应用
   在 src 目录 index.js 增加以下内容

```js
import { registerMicroApps, start } from "qiankun";

registerMicroApps([
  {
    name: "sub-react",
    entry: "//localhost:7002",
    container: "#root",
    activeRule: "/my-sub-react",
  },
]);
// 启动 qiankun
start();
```

### 微应用配置

#### 微应用 sub-react

1. 用脚手架创建工程

```
$ yarn create react-app xxx-demo

# or

$ npx create-react-app xxx-demo
```

2. 在 src 目录新增 public-path.js

```
if (window.__POWERED_BY_QIANKUN__) {
   // eslint-disable-next-line no-undef
    __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
  }
```

3. 入口文件修改 src/index.js

```js
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import reportWebVitals from "./reportWebVitals";
import "./index.css";

function render(props) {
  const { container } = props;
  ReactDOM.render(
    <App />,
    container
      ? container.querySelector("#root")
      : document.querySelector("#root")
  );
}

if (!window.__POWERED_BY_QIANKUN__) {
  render({});
}

export async function bootstrap() {
  console.log("[react16] react app bootstraped");
}

export async function mount(props) {
  console.log("[react16] props from main framework", props);
  render(props);
}

export async function unmount(props) {
  const { container } = props;
  ReactDOM.unmountComponentAtNode(
    container
      ? container.querySelector("#root")
      : document.querySelector("#root")
  );
}

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

4. 修改 webpack 配置

```
$ yarn add react-app-rewired

# or

$ npm i -D react-app-rewired
```

在 src 目录下新增 config-overrides.js 文件

```
const { name } = require('./package');

  module.exports = {
    webpack: config => {
      config.output.library = `${name}-[name]`;
      config.output.libraryTarget = 'umd';
      config.output.chunkLoadingGlobal = `webpackJsonp_${name}`; //output.jsonpFunction 已更新为 => output.chunkLoadingGlobal
      return config;
    },
    devServer: (configFunction) => {
      return (proxy, allowedHost) => {
        const config = configFunction(proxy, allowedHost);
        config.historyApiFallback = true;
        config.open = false;
        config.hot = false;
        config.static = false;
        config.liveReload = false;
        config.headers = {
          'Access-Control-Allow-Origin': '*',
        };
        return config;
      }
    }
  }
```

### 部署
