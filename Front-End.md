<!-- MarkdownTOC -->

- [待整理](#待整理)
  - [工作环境配置](#工作环境配置)
    - [1. Ubuntu14.04 服务器上安装 NodeJS](#1-ubuntu1404-服务器上安装-nodejs)
    - [2. Ubuntu 14.04 安装 node v4](#2-ubuntu-1404-安装-node-v4)
    - [3. glup安装](#3-glup安装)
    - [4. webpack + React 镜像安装](#4-webpack--react-镜像安装)
  - [工具使用](#工具使用)
    - [1. glup](#1-glup)
    - [2. React基础讲解](#2-react基础讲解)
    - [3. webpack](#3-webpack)
    - [4. webpack + React](#4-webpack--react)
    - [5. webpack + css](#5-webpack--css)
    - [6. react-hot-boilerplate 安装](#6-react-hot-boilerplate-安装)
    - [7. SVG](#7-svg)
    - [8. React组件复用](#8-react组件复用)

<!-- /MarkdownTOC -->


<a name="待整理"></a>
# 待整理

<a name="工作环境配置"></a>
## 工作环境配置

<a name="1-ubuntu1404-服务器上安装-nodejs"></a>
#### 1. Ubuntu14.04 服务器上安装 NodeJS
>0. 添加 PPA
`$ curl -sL https://deb.nodesource.com/setup | sudo bash -`
1. 安装 PPA 中提供的nodejs 和 npm
`$ sudo apt-get install nodejs`
2. 安装编译环境
由于 npm 装包的时候，有些 node 的包是需要在本地编译的，所以还要保证系统上有编译环境，很简单，只需要安装
`$ sudo apt-get install build-essential`

>参考资料: [How To Install Node.js on an Ubuntu 14.04 server](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-an-ubuntu-14-04-server)

<a name="2-ubuntu-1404-安装-node-v4"></a>
#### 2. Ubuntu 14.04 安装 node v4

    打开终端输入

    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.27.1/install.sh | bash

    退出 shell 重新登录，执行

    nvm ls-remote

    可以看到最近的 node 版本是 4.1.1 ，执行安装

    nvm install 4.1.1

    这样会报警告

    ···
    nvm is not compatible with the npm config "prefix" option: currently set to "/home/peter/.npm-global" Run npm config delete prefix or nvm use --delete-prefix v4.1.1 to unset it.
    ···

    这是由于咱们自己做的 prefix 造成的。解决方法

    mv .npmrc npmrc
    nvm install 4.1.1
    mv npmrc .npmrc

    最后

    node -v

    可以看到系统上的 node 版本已经是 4.1.1 了

<a name="3-glup安装"></a>
#### 3. glup安装

    npm get prefix        //获取Nodejs安装位置信息

      默认情况下，npm 全局装包会安装到 /usr/local/ 之中。
      默认情况下，如果不用 sudo ，很可能会报权限错误。
      最好的方式是修改一下全局安装的位置。

    mkdir ～/.npm-global  //新建安装位置
    npm config set prefix '~/.npm-global'  //设置npm全局安装位置

    npm i -g gulp         //安装gulp
    cd .npm-global/lib/node_modules        //gulp安装位置
    cd .npm-global/bin    //gulp 执行文件的位置，需要在.bashrc或.profile中添加 export PATH=~/.npm-global/bin:$PATH
    source .bashrc        //修改后加载

    gulp          //确认gulp安装是否成功
    which gulp    //确认gulp安装位置

<a name="4-webpack--react-镜像安装"></a>
#### 4. webpack + React 镜像安装

    npm i -g webpack     //全局安装webpack，如果失败可以使用淘宝镜像，方法如下

    在～/.bashrc中添加
    #alias for cnpm
    alias cnpm="npm --registry=https://registry.npm.taobao.org \
    --cache=$HOME/.npm/.cache/cnpm \
    --disturl=https://npm.taobao.org/dist \
    --userconfig=$HOME/.cnpmrc "

    source .bashrc       //添加后运行
    vim .cnpmrc          //打开cnpmrc，输入 prefix=~/.npm-global
    source .cnpmrc       //添加后运行

    cnpm i -g webpack    //使用淘宝镜像进行全局安装

    测试：新建一个文件夹，进入后输入
    npm init             //一路回车，生成 package.json 文件
    cnpm i -g react react-dom        //全局安装，查看返回信息 react@0.14.0 /home/godi13/.npm-global/lib/node_modules/react 正确
    cnpm i --save react react-dom    //局部安装，查看返回信息 react@0.14.0 node_modules/react 正确

>[淘宝npm镜像](http://npm.taobao.org/)  
>[webpack学习资料](https://fakefish.github.io/react-webpack-cookbook/)

***

<a name="工具使用"></a>
## 工具使用

<a name="1-glup"></a>
#### 1. glup

    进入项目文件夹下，在项目内安装gulp

    gulp -v                  //查看gulp版本信息
    npm init                 //一路回车确认，保留默认设置即可，得到 package.json
    npm i gulp --save-dev    //将gulp信息保留到package.json，同时会生成node_modules文件夹
    vim package.json         //可以在最下面看到gulp版本信息
    vim .gitignore           //输入node_modules/
    npm i gulp-sass -D       // i 相当于install，-D 相当于--save-dev，gulp-sass 依赖于 libsass

```js
创建 gulpfile.js 文件

var gulp = require('gulp');
var sass = require('gulp-sass');

gulp.task('sass',function(){    //任务名’sass‘
  gulp.src('styles/main.scss')  //被处理的文件
      .pipe(sass())             //传递给sass()进行处理
      .pipe(gulp.dest('css'));  //输出的位置
  });
```
    gulp sass   //执行任务名为'sass'的命令，会新建文件夹css，里面有main.css

###### 自动更新sass变动

```js
在 gulpfile.js 文件中添加

gulp.task('watch',function(){
  gulp.watch('styles/*.scss',['sass']);  //styles下的.scss文件有改动便执行sass命令
});
```
    gulp watch  //开始自动更新

###### 在gulp出错时不中止watch的方法

```js
function handleError(err) {
  console.log(err.toString());
  this.emit('end');
}

      .on('error', handleError)  //加入.pipe(sass())下面即可
```

###### 安装其他的gulp插件方法，如 autoprefixer

    npm i gulp-autoprefixer -D    //先在终端安装插件

```js
在 gulpfile.js 文件对应的位置上添加

var prefix = require('gulp-autoprefixer');

      .pipe(prefix())            //prefix()可以设置支持的浏览器版本，具体的看官方文档
```

<a name="2-react基础讲解"></a>
#### 2. React基础讲解

>[React中文教程](http://wiki.jikexueyuan.com/project/react/)  
>[React学习笔记](React.md)

0. 命名组件时，首字母要大写  
0. [国内cdn](http://www.bootcdn.cn/react/) 复制`//cdn.bootcss.com/react/0.14.0-rc1/react.js` 粘贴后添加`http:` 不用启动服务器文件
0. _JSXTransformer.js_ 仅在开发测试阶段使用，上线之前需要预编译
0. [HTML to JSX Compiler](http://facebook.github.io/react/html-jsx.html) 转换 _HTML_ 到 _JSX_
0. 父子嵌套时，父元素要注意包裹才可生效，出问题可以看一下Chrome开发者工具中报的什么错误

>原版

```html
<script src="http://cdn.bootcss.com/react/0.14.0-rc1/react.js"></script>

<div id="app"></div>
<script>
var HelloWorld = React.createClass({
  render: function(){
    return (
      React.createElement("div", null,
        "Hello World!"
      )
    )
  }
});
React.render(React.createElement(HelloWorld, null), document.getElementById('app'));
</script>
```

>JSX版

```html
<script src="http://cdn.bootcss.com/react/0.14.0-rc1/react.js"></script>
<script src="http://cdn.bootcss.com/react/0.14.0-beta3/JSXTransformer.js"></script>

<div id="app"></div>
  <script type="text/jsx">
    var HelloWorld = React.createClass({
      render: function(){
        return (
            <div> Hello World </div>
            )
          }
    });
    React.render(<HelloWorld />, document.getElementById('app'));
</script>
```

>父子嵌套

```html
<script src="http://cdn.bootcss.com/react/0.14.0-rc1/react.js"></script>
<script src="http://cdn.bootcss.com/react/0.14.0-beta3/JSXTransformer.js"></script>

<div id="app"></div>
<script type="text/jsx">
  var Child= React.createClass({
    render: function(){
      return (
          <div> Child </div>
          )
        }
  });
  var Parent = React.createClass({
    render: function(){
      return (
          <div>
            <div> Hello World </div>
            <Child />
          </div>
          )
        }
  });
  React.render(<Parent />, document.getElementById('app'));
</script>
```
>参考视频教程： *HappyPeter* 老师的 [好多视频第164期](http://haoduoshipin.com/v/164)

<a name="3-webpack"></a>
#### 3. webpack

    npm init    //初始化，生成package.json文件，一路回车即可

    npm i --save-dev webpack    //局部安装webpack

  >打开subl，创建 _webpack.config.js_ 配置文件，进入[webpack网站](http://webpack.github.io/docs/tutorials/getting-started/)，复制如下代码：

```js
module.exports = {
    entry: "./entry.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }
};
```
>测试：只是打捆js文件，可以删除暂时用不到的代码，修改 entry.js 为 main.js，如下：

```js
module.exports = {
    entry: "./main.js",         //修改为自己习惯名字
    output: {
        path: __dirname,        //输出位置，当前文件夹下
        filename: "bundle.js"   //输出文件名
    }
};
```
>分别创建main.js，hello.js添加如下内容

```js
//main.js
require("./hello.js");          //此处要添加路径./，否则会报错
console.log("I am main.js");

//hello.js
console.log("I am hello.js");
```
>完成后在终端执行 `webpack` ，生成 `bundle.js` 文件，执行 `node bundle.js` 确认结果

<a name="4-webpack--react"></a>
#### 4. webpack + React

>删除 _index.html_ 中其他的js，加入

```html
<script src="bundle.js"></script>
```

>创建 _webpack.config.js_ 配置文件，输入以下内容

```js
module.exports = {
    entry: "./main.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.js$/, loader: "babel" }  //检测文件后缀名为.js，打捆js文件
        ]
    }
};
```
    cnpm i --save-dev babel-loader   //局部安装babel-loader完成后，可以把index.html中的JSXTransformer.js路径删除
    cnpm i --save react react-dom    //局部安装react完成后，可以把index.html中的React.js路径删除

###### 实际案例修改过程

>在各个组件中添加如下代码

```js
var React = require('react');        //调用react

module.exports = 组件名称;            //将组件变成模块，保证 main.js 可以 require 进来
```
>在 _App.js_ 中添加

```js
var React = require('react');
var Action = require('./Action.js');  //路径
var Haoduo = require('./Haoduo.js');  //路径
var Books = require('./Books.js');    //路径

module.exports = App;
```
>新建 _main.js_ 文件，添加

```js
var React = require('react');
var ReactDOM = require('react-dom');                        //React v0.14 建议增加ReactDOM
var App = require('./components/App.js');                   //注意要有路径

ReactDOM.render(<App />, document.getElementById('app'));  //React v0.14 建议改为ReactDOM.render
```
>以上都修改完成后，执行 `webpack`

<a name="5-webpack--css"></a>
#### 5. webpack + css

>删除 _gulpfile.js_  
删除 _package.json_ 中 _gulp_ 相关工具  
安装以下loader

    cnpm i -D style-loader css-loader sass-loader autoprefixer-loader

>在 _webpack.config.js_ 文件中添加 _loader_

```js
module.exports = {
    entry: "./main.js",
    output: {
        path: __dirname,
        filename: "bundle.js"
    },
    module: {
        loaders: [
            { test: /\.js$/, loader: "babel" },                         //注意这要有一个逗号！！
            { test: /\.scss$/, loader: "style!css!autoprefixer!sass" }  //检测后缀为 .scss 的文件
        ]
    }
};
```
>现在可以直接在 _main.js_ 中 _require_ 非 _js_ 文件了，添加

```js
require('./styles/main.scss');
```
>现在可以删除 _index.html_ 中的

```html
  <link rel="stylesheet" type="text/css" href="css/main.css">
```

    webpack --watch   //效果类似于 gulp watch

>__最终只需要上传 _index.html_ 和 _bundle.js_ 文件即可__

<a name="6-react-hot-boilerplate-安装"></a>
#### 6. react-hot-boilerplate 安装

>* 从 github 下载 [_react-hot-boilerplate_](https://github.com/gaearon/react-hot-boilerplate)
* 进入文件夹输入 `npm i` 进行安装
* 输入 `npm start` 运行服务，在浏览器输入 `localhost:3000` 即可

```js
//webpack.config.js 文件配置
var path = require('path');
var webpack = require('webpack');

module.exports = {
  devtool: 'eval',          // eval 代表提高编译速度
  entry: [
    'webpack-dev-server/client?http://localhost:3000',
    'webpack/hot/only-dev-server',
    './src/index'           // 根据项目名称此处修改为 './src/main.js'
  ],
  output: {
    path: path.join(__dirname, 'dist'),  //编译输出位置
    filename: 'bundle.js',
    publicPath: '/static/'  //指定内存位置，index.html主页上的 bundle.js路径也要相应修改为 <script src="/static/bundle.js"></script>
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ],
  module: {
    loaders: [
    { test: /\.js$/, loaders: ['react-hot', 'babel'], include: path.join(__dirname, 'src') },
    //注意此处为loaders，并且所有文件夹都要放着src下面，比如 components,main.js,styles/ 之类
    { test:/\.scss$/, loader: "style!css!autoprefixer!sass" }
    ]
  }
};
```
    cnpm i -D react-hot-loader webpack-dev-server  //需要安装

```js
//在项目中创建 server.js
var webpack = require('webpack');
var WebpackDevServer = require('webpack-dev-server');
var config = require('./webpack.config');

new WebpackDevServer(webpack(config), {
  publicPath: config.output.publicPath,
  hot: true,
  historyApiFallback: true
}).listen(3000, 'localhost', function (err, result) {
  if (err) {
    console.log(err);
  }

  console.log('Listening at localhost:3000');
});
```
    node server.js    //开发模式下使用的命令

    webpack           //调试完成后输入,可以在 dist 目录下可以找到 bundle.js

>参考资料： [webpack.config.js 参数详解](http://gaearon.github.io/react-hot-loader/getstarted/)  
>参考资料： [非 _React_ 构架下使用热模块替换的方法](http://christianalfoni.github.io/react-webpack-cookbook/Automatic-browser-refresh.html)

<a name="7-svg"></a>
#### 7. SVG

    cnpm i -D react-inlinesvg

```js
var Isvg = require('react-inlinesvg');

<Isvg src="/path/to/myfile.svg"></Isvg>
```

>参考资料： [react-inlinesvg](https://github.com/matthewwithanm/react-inlinesvg)

<a name="8-react组件复用"></a>
#### 8. React组件复用

>React支持从父组件向子组件传递属性值

```js
//Button.js
var React = require('react');
var classNames = require('classnames');
var Button = React.createClass({
  render: function() {
    var classes = classNames({
      button: true,
      green: this.props.isGreen,
      pink: this.props.isPink
    })
    return (
          <a href="#" className={classes} >{this.props.text}</a>    //classes 会最终接受到 "button green" 或 "button pink"
      );
    }
  });
module.exports = Button;

//Action.js
var React = require('react');
var Button = require('./Button.js');
var Action = React.createClass({
  render: function() {
    return (
        <div className="action clearfix">
          <Button isGreen={true} text={'Join'}/>    //通过传递 isGreen 还是 isPink 来控制
        </div>
      );
    }
  });
module.exports = Action;
```

***
