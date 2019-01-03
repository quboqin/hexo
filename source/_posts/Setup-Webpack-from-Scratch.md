---
title: Setup Webpack from Scratch
date: 2018-12-31 16:25:26
tags: [__dirname, path, ../, ./, require, readFileSync]
categories:
  - Nodejs
---

## What is the difference between __dirname and ./ in Nodejs
Three different situations need to be considered. 
1. In Nodejs, `__dirname` and `__filename` are always the directories in which these scripts reside.
2. By contrast, `./` and `../` are the directories from which you run these scripts in Nodejs environment when you use libraries like `path` and `fs`.
3. The paths in `require` are alway relative to these files.
FOr more detail information, here is the [reference link](https://stackoverflow.com/questions/8131344/what-is-the-difference-between-dirname-and-in-node-js) on Stackoverflow.   

I give an example below, our directory is like this:
```
webpack-learning/src
├── app
└── server
    ├── index.js
    └── utils
        └── pathtest.js
```
and the `server/utils/pathtest.js` contains
```javascript
const chalk = require('chalk')

const path = require('path')
const fs = require('fs')

module.exports = function () {
  console.log(chalk.yellow('__filename = %s \n__dirname = %s', __filename, path.resolve(__dirname)))
  console.log(chalk.yellow('. = %s', path.resolve('.')))

  console.log(require('../../../assets/media/config.json'))
  console.log(chalk.yellow(fs.readFileSync(path.resolve(__dirname, '../../../assets/media/sometext.txt'), 'utf8')))
  console.log(chalk.red.bgYellow('the next readFileSync() will throw out an exception if the working directory is not on the top of src!'))
  console.log(chalk.red.bgYellow(fs.readFileSync('./assets/media/sometext.txt', 'utf8')))
}
```
the `server/index.js` is
```javascript
require('./utils/pathtest')()
```
When we run the script in the parent directory of `src`,
```shell
➜  webpack-learning git:(testpath) node src/server/index.js
```
we get this result in the console.
```shell
__filename = %s
__dirname = %s /Users/qinqubo/Magicefire/Playground/webpack-learning/src/server/utils/pathtest.js /Users/qinqubo/Magicefire/Playground/webpack-learning/src/server/utils
. = %s /Users/qinqubo/Magicefire/Playground/webpack-learning
{ hello: 'world' }
some txt
the next readFileSync() will throw out an exception if the working directory is not on the top of src!
some txt
```
If we run this script in other directory, the second `readFileSync` function call will throw out an error in the console like this,
```
...
some txt
the next readFileSync() will throw out an exception if the working directory is not on the top of src!
fs.js:646
  return binding.open(pathModule._makeLong(path), stringToFlags(flags), mode);
                 ^

Error: ENOENT: no such file or directory, open './assets/media/sometext.txt'
    at Object.fs.openSync (fs.js:646:18)
    at Object.fs.readFileSync (fs.js:551:33)
    at module.exports (/Users/qinqubo/Magicefire/Playground/webpack-learning/src/server/utils/pathtest.js:13:37)
    at Object.<anonymous> (/Users/qinqubo/Magicefire/Playground/webpack-learning/src/server/index.js:1:90)
    at Module._compile (module.js:652:30)
    at Object.Module._extensions..js (module.js:663:10)
    at Module.load (module.js:565:32)
    at tryModuleLoad (module.js:505:12)
    at Function.Module._load (module.js:497:3)
    at Function.Module.runMain (module.js:693:10)
```
You can examine the differences between the two `readFileSync` function calls. Because the second one use `../` as its path, and these `./` and `../` are depended on the directory where you run the script. So the second one got an error.

I pushed the example code on [Github](https://github.com/quboqin/webpack-learning), you can check out on the `test-path` branch.

## The Paths in the `webpack.config.js`
#### Common Options
The `Webpack` is a compiler which reads a file specified by `config.entry` and recursively builds a dependency graph and then packages all these artifacts into a group of bundles(or a single bundle). 

<img src="/Setup-Webpack-from-Scratch/webpack-module-bundler.png" width="50%" margin="auto">

So a bunch of options of `config` in the `webpack.config.js` are about how to find the the **input files** in our project folder, and the others are used to figure out where the `Webpack` keep the **output files**. Some of these properties give absolute paths, some of them are relative to other options.

The table below gives some examples about the **input** options

| name | role | default value | dependency | 
| - | - | - | - | - |
| context | The base directory, an absolute path, for resolving entry points and loaders from configuration. | The default value is `process.cwd()` but it's recommended to replace it with an absolute address `context: path.resolve(__dirname, 'app')` |  |
| entry | The point or points to enter the application | | `context` If entry is an absolute address, the context will be ignored |
| template(html-webpack-plugin) | require path to the template. | '' | `context` |

When we discuss the **output**, we need to seperate 2 different situations. If we just let the `Webpack` package the bundle for us, then the `Webpack` create all **output files** on the disk. But
when we use the `webpack-dev-server`, the web server will mount these **output files** in the memory. The options are somewhat different.

Now we use `webpack webpack --config ./build/webpack.config.js` to create a distribute folder, here is the options we need to specify.

| name | role | default value | dependency | 
| - | - | - | - | - |
| output.path | The output directory as an absolute path. | | |
| output.filename | This option determines the name of each output bundle. | | The bundle is written to the directory specified by the `output.path` option |
| outputPath(file-loader) | | | `output.path` |
| filename(HtmlWebpackPlugin) | | | `output.path` |

When we start the `webpack-dev-server`, some of the options are different.

| name | role | default value | dependency | 
| - | - | - | - | - |
| devServer.publicPath | Its role is same as the role of `output.path`. The differency is that it is mounted in the memory and as part of URL address which only be accessed from a web browser through the `express` server. | '/' | |
| devServer.contentBase | Tell the server where to serve content from. This is only necessary if you want to serve static files. devServer.publicPath will be used to determine where the bundles should be served from, and takes precedence. **I think this option is useless and confused**. Maybe when some resources are not packaged into our bundle, but can be accessed under the run time. In this case, maybe we would use this option. | | |

#### The Most Confused Part, `publicPath`
The most confused parts in the config of the `Webpack` are the two `publicPath`s. One is the `devServer.publicPath` we mention it before. The other is the `output.publicPath`. The latter in most case is a placeholder used by `loaders` and/or `plugins` to prefix(update) the URLs in the modules. In the `production` mode, we usually use this option to specify the CDN address, so we don't need to manually update the URLs in all the files to point to the CDN. 

```javascript
module.exports = {
  //...
  output: {
    // One of the below
    publicPath: 'https://cdn.example.com/assets/', // CDN (always HTTPS)
    publicPath: '//cdn.example.com/assets/', // CDN (same protocol)
    publicPath: '/assets/', // server-relative
    publicPath: 'assets/', // relative to HTML page
    publicPath: '../assets/', // relative to HTML page
    publicPath: '', // relative to HTML page (same directory)
  }
}
```

But, here is alway a **But**. When the `output.publicPath` is not a CDN address, it will influence the output result when we start the `webpack-dev-server`. I’m really bad at googling staff, I tried to find some articles, but they didn't meet my expectations. And the document of `Webpack` is also terrible.

There are common mistakes we usually meet.
1. If we assign the values for `devServer.publicPath` and `output.publicPath`, but the values are different. The html cabn be loaded successfully, but the bundle and the other resource can not be accessed.
```javascript
const config = {
  output: {
    publicPath: '/notdist/',

    path: path.resolve(__dirname, '../dist'),
    filename: '[name].bundle.js'
  },
  ...
  devServer: {
    // static files served from here
    publicPath: '/dist/',

    port: 2000,
    stats: 'errors-only',
    open: true
  },
}
```
<img src="/Setup-Webpack-from-Scratch/webpack-error-1.png" width="80%" margin="auto">

2. If both of these `publicPath`s are not given values, the default value of the `devServer.publicPath` is '/', and the default value of the `output.publicPath` is ''. That will be ok. But if we assigned a value to the `output.publicPath`, but didn't give a value to the `devServer.publicPath`, the `devServer.publicPath` will use the `output.publicPath` value!
```javascript
const config = {
  output: {
    publicPath: '/dist/',

    path: path.resolve(__dirname, '../dist'),
    filename: '[name].bundle.js'
  },
  ...
  devServer: {
    // static files served from here
    // publicPath: '/dist/',

    port: 2000,
    stats: 'errors-only',
    open: true
  },
}
```
```shell
> NODE_ENV=development webpack-dev-server --config ./build/webpack.config.js

clean-webpack-plugin: /Users/qinqubo/Magicefire/Playground/webpack-learning/dist has been removed.
ℹ ｢wds｣: Project is running at http://localhost:2000/
ℹ ｢wds｣: webpack output is served from /dist/
⚠ ｢wdm｣:
ℹ ｢wdm｣: Compiled with warnings.
```
<img src="/Setup-Webpack-from-Scratch/webpack-error-2.png" width="80%" margin="auto">
It is not an error, but it is very confused:(

3. The example before, we start the `output.publicPath` with the '/', so it is a server-relative path. So far everything is fine. But if we set this option using an address which is relative to HTML page, like 'dist/'
```javascript
const config = {
  output: {
    publicPath: 'dist/',

    path: path.resolve(__dirname, '../dist'),
    filename: '[name].bundle.js'
  },
  ...
  devServer: {
    // static files served from here
    // publicPath: '/dist/',
    // open app in localhost:2000
    port: 2000,
    stats: 'errors-only',
    open: true
  },
}
```
```shell
> NODE_ENV=development webpack-dev-server --config ./build/webpack.config.js

clean-webpack-plugin: /Users/qinqubo/Magicefire/Playground/webpack-learning/dist has been removed.
ℹ ｢wds｣: Project is running at http://localhost:2000/
ℹ ｢wds｣: webpack output is served from /dist/
ℹ ｢wdm｣: wait until bundle finished: /dist/index.html
⚠ ｢wdm｣:
ℹ ｢wdm｣: Compiled with warnings.
```
`loaders` and `plugins` will update URLs with the html-relative path, here is the 'dist/', but webpack output is served from '/dist/', we can access the html file at `http://localhost:2000/dist/index.html`, The linkers in this html file are pointing to the wrong addresses, such as `src="dist/assets/media/webpacklogo.png"` or `src="dist/app.bundle.js"` It is a relative path to the html file, but the html file and the bundle filea are already in the `dist`, and there is no sub-folder called `dist`
<img src="/Setup-Webpack-from-Scratch/webpack-error-3.png" width="80%" margin="auto">
<img src="/Setup-Webpack-from-Scratch/webpack-error-4.png" width="80%" margin="auto">

It is a disaster. My suggestion is avoiding using the `derServer.publicPath` and not using `output.publicPath` under the `development` mode. The only purpose of the `output.publicPath` is setting CDN address under the `production` mode.
```javascript
const env = process.env.NODE_ENV      

const config = {
  ...

  output: {
    // absolute path declaration
    publicPath: env === 'development' ? '' : 'https://cdn.example.com/assets/',
    path: path.resolve(__dirname, '../dist'),
    filename: '[name].bundle.js'
  },

  ...
}
```

#### The path of CleanWebpackPlugin
If we use `CleanWebpackPlugin` to remove the 'dist' folder, we must set the `root` value in its options when the config file is not in the root of the project.

```javascript 
plugins: [
 // cleaning up only 'dist' folder
 new CleanWebpackPlugin(['dist'], {
   root: path.resolve(__dirname, '../'),
 }),
```

You can check out on the `test-publicpath` branch on [Github](https://github.com/quboqin/webpack-learning).






