# nodejs/npm 安装

官网

https://nodejs.org/en

>win mac linux 版本
>现在是 20 版本，稳定版好像是18

安装很简单，下一步、下一步即可

安装完成后，记得看下是否加到 环境变量中

查看 node 版本：

> node \-v

node 安全成功 后，会自带 npm

> npm \-v

查看 npm 配置信息
> npm config ls

更改 安装目录
> npm config set prefix "D:\\nodejs\\node\_global"

查看已安装的包：

> npm install webpack \-\-save\-dev

# 创建一个新项目

cd /data/nodejs
mkdir projceName
npm init

然后就是一顿回车即可
会在根目录多一个 package.json
vim package.json

```
{
    "name": "zjsframword",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "type": "module",
    "scripts": {
    "   test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
        "dependencies": {
        "crypto-js": "^4.1.1",
        "protobufjs": "^7.2.4",
        "ws": "^8.13.0",
        "yamljs": "^0.3.0"
    }
}
```

分析：

1. 大概就是项目的基础
2. 依赖哪些库
3. scripts 带些脚本，如：npm run build / npm run serve 等

# webpack

npm i @types/node \-\-save\-dev

npm install stream\-http

npm install path\-browserify

> 上面这3个包，因为 webpack 打包时报错

npm install \-g webpack

npm install \-g webpack\-cli

看下版本信息

> webpack \-v

创建配置文件

> touch webpack.config.js

编辑配置文件

> vim webpack.config.js

创建目录/文件

```
mkdir src
cd src
touch index.js
```

简单测试一下：

> webpack.cmd ./src/index.js \-\-output\-path ./dist \-\-mode=development

