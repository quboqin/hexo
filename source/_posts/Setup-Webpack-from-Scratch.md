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

I pushed the example code on [Github](https://github.com/quboqin/webpack-learning), you can check out on the `pathtest` branch.