---
title: hexo提交新文章错误
date: 2017-03-19 10:33:53
tags: [hexo]
categories: hexo
---
# hexo提交新文章出错
&emsp;&emsp;提交文章的时候报了下面这个错误，
<!--more-->
```
ERROR Process failed: _posts/数组中出现次数超过一半的数字.md
YAMLException: can not read a block mapping entry; a multiline key may not be an                                             implicit key at line 5, column 1:

    ^
    at generateError (F:\Myblog\hexo\node_modules\hexo\node_modules\js-yaml\lib\                                            js-yaml\loader.js:162:10)
    at throwError (F:\Myblog\hexo\node_modules\hexo\node_modules\js-yaml\lib\js-                                            yaml\loader.js:168:9)
    at readBlockMapping (F:\Myblog\hexo\node_modules\hexo\node_modules\js-yaml\l                                            ib\js-yaml\loader.js:1045:9)
    at composeNode (F:\Myblog\hexo\node_modules\hexo\node_modules\js-yaml\lib\js                                            -yaml\loader.js:1331:12)
    at readDocument (F:\Myblog\hexo\node_modules\hexo\node_modules\js-yaml\lib\j                                            s-yaml\loader.js:1493:3)
    at loadDocuments (F:\Myblog\hexo\node_modules\hexo\node_modules\js-yaml\lib\                                            js-yaml\loader.js:1549:5)
    at Object.load (F:\Myblog\hexo\node_modules\hexo\node_modules\js-yaml\lib\js                                            -yaml\loader.js:1566:19)
    at parseYAML (F:\Myblog\hexo\node_modules\hexo\node_modules\hexo-front-matte                                            r\lib\front_matter.js:80:21)
    at parse (F:\Myblog\hexo\node_modules\hexo\node_modules\hexo-front-matter\li                                            b\front_matter.js:56:12)
    at F:\Myblog\hexo\node_modules\hexo\lib\plugins\processor\post.js:52:18
    at tryCatcher (F:\Myblog\hexo\node_modules\hexo\node_modules\bluebird\js\rel                                            ease\util.js:16:23)
    at Promise._settlePromiseFromHandler (F:\Myblog\hexo\node_modules\hexo\node_                                            modules\bluebird\js\release\promise.js:509:35)
    at Promise._settlePromise (F:\Myblog\hexo\node_modules\hexo\node_modules\blu                                            ebird\js\release\promise.js:569:18)
    at Promise._settlePromise0 (F:\Myblog\hexo\node_modules\hexo\node_modules\bl                                            uebird\js\release\promise.js:614:10)
    at Promise._settlePromises (F:\Myblog\hexo\node_modules\hexo\node_modules\bl                                            uebird\js\release\promise.js:693:18)
    at Promise._fulfill (F:\Myblog\hexo\node_modules\hexo\node_modules\bluebird\                                            js\release\promise.js:638:18)
    at PromiseArray._resolve (F:\Myblog\hexo\node_modules\hexo\node_modules\blue                                            bird\js\release\promise_array.js:126:19)
    at PromiseArray._promiseFulfilled (F:\Myblog\hexo\node_modules\hexo\node_mod                                            ules\bluebird\js\release\promise_array.js:144:14)
    at PromiseArray._iterate (F:\Myblog\hexo\node_modules\hexo\node_modules\blue   
```
仔细排查之后，发现是文章开头的categroies的冒号，写成了中文的冒号。额。。。