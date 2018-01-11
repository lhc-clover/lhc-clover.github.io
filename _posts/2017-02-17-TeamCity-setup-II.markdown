---
layout: post
title:  "持续集(打)成(包)平台TeamCity 续"
date:   2017-02-17 9:00:00
category: Tools
---
<br />
自[上一篇](https://lhc-clover.github.io/tools/2016/03/15/TeamCity-setup.html)中提到的持续集(打)成(包)平台投入使用后，出于业务的需要或更自动化(懒)的需求，又在上面增加了一些其他的功能：

- 向fir.im自动发布
- 使用CheckStyle代码检查

### fir.im发布
[fir.im](https://fir.im)是一个应用内测托管平台，能将App发布成可扫码下载。并不是应用市场，但用它能与QA更好的协作。   

可自动发布的前提是可发布，也就是获取API token啥的，略。

首先配置gradle。根据官网的[文档](http://blog.fir.im/gradle/)在gradle脚本中加入相关依赖和API token。完成后再编译时就会发现多出了以`publishApk`开头的task。这个task与打包的buildType有关，如buildType叫release，那么多出的task就叫`publishApkRelease`。注意，文档也有提到，必须打开zipAlign才会生成task，release默认是打开的。   

