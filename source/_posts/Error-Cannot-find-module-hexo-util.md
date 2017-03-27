---
title: 'Error: Cannot find module ''hexo-util'''
date: 2017-03-27 16:25:03
categories:
- Hexo
tags:
- hexo
---

升级NexT主题以后，执行`hexo clean`，出错：

```shell
✗ hexo clean         
ERROR Script load failed: themes/next/scripts/tags/exturl.js
Error: Cannot find module 'hexo-util'
    at Function.Module._resolveFilename (module.js:325:15)
    at Function.Module._load (module.js:276:25)
    at Module.require (module.js:353:17)
    at require (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/lib/hexo/index.js:213:21)
    at /Users/lisanlai/lisanlai.github.io/themes/next/scripts/tags/exturl.js:8:12
    at /Users/lisanlai/lisanlai.github.io/node_modules/hexo/lib/hexo/index.js:229:12
    at tryCatcher (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:502:31)
    at Promise._settlePromise (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:559:18)
    at Promise._settlePromise0 (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:604:10)
    at Promise._settlePromises (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:683:18)
    at Promise._fulfill (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:628:18)
    at Promise._resolveCallback (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:423:57)
    at Promise._settlePromiseFromHandler (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:514:17)
    at Promise._settlePromise (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:559:18)
    at Promise._settlePromise0 (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:604:10)
    at Promise._settlePromises (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:683:18)
    at Promise._fulfill (/Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/promise.js:628:18)
    at /Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/bluebird/js/release/nodeback.js:42:21
    at /Users/lisanlai/lisanlai.github.io/node_modules/hexo/node_modules/hexo-fs/node_modules/graceful-fs/graceful-fs.js:78:16
    at FSReqWrap.readFileAfterClose [as oncomplete] (fs.js:380:3)
INFO  Deleted database.

```

**解决办法：重新安装hexo-util模块 **

`npm install -- save-dev hexo-util`