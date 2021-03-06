### 目录

[启动](#启动)
[主应用配置](#主应用配置)
[微应用配置](#微应用配置)
[部署](#部署)

### 启动

1. 先启动微应用

```
cd sub-react

npm i or yarn

npm run start or yarn start
```

2. 再启用主应用，主应用启动同上

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
    entry: "//localhost:7002/sub-react/",
    container: "#subapp-viewport",
    activeRule: "/child-sub-react",
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
      ? container.querySelector("#sub-react-root")
      : document.querySelector("#sub-react-root")
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
      ? container.querySelector("#sub-react-root")
      : document.querySelector("#sub-react-root")
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

```js
const { name } = require("./package");

module.exports = {
  webpack: (config) => {
    config.output.library = `${name}-[name]`;
    config.output.libraryTarget = "umd";
    config.output.chunkLoadingGlobal = `webpackJsonp_${name}`; //output.jsonpFunction 已更新为 => output.chunkLoadingGlobal,
    config.output.publicPath = `/sub-react/`;
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
        "Access-Control-Allow-Origin": "*",
      };
      return config;
    };
  },
};
```

修改 package.json 中的 scripts

```js
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-app-rewired eject"
  },
```

5. 设置 history 模式路由的 base

```tsx
<BrowserRouter
  basename={window.__POWERED_BY_QIANKUN__ ? "/child-sub-react" : "/"}
>
  <App />
</BrowserRouter>
```

### 部署

1. 主应用和微应用部署在不同的服务器，使用 Nginx 代理访问

- 将主应用的工程代码放到服务器任意目录下
- 在主应用根目录执行`sh deploy.sh`,脚本会安装工程依赖，打包，并将 build 包放置`/etc/nginx/build`目录下
- 主应用 nginx 配置
  进入 nginx 配置目录`cd /etc/nginx/ `，查看 nginx 配置文件`vi nginx.conf`，重点配置内容如下：

```
  server {
    listen       7002;
    server_name  10.83.20.125;

    location / {
                root   /etc/nginx/build;
                index  index.html index.htm;
    }
    location /sub-react/ {
        proxy_pass http://10.83.20.126:7002/sub-react/;
        proxy_set_header Host 10.83.20.125:7002;
    }
  }
```

微应用 nginx 配置

微应用部署同上，重点配置内容如下：

```
 server {
    listen       7002;
    server_name  10.83.20.126;

    location / {
        root   /etc/nginx/sub;
        # 注意这个下面这个index配置
        # 微应用build完的静态资源包路径在/etc/nginx/sub/sub-react目录
        index  index.html sub-react/index.html;
    }
}
```
