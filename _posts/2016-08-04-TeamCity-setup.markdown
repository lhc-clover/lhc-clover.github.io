---
layout: post
title:  "持续集(打)成(包)平台TeamCity"
date:   2016-03-15 15:00:00
category: Tools
---
<br />
由于我的坚持，组内的工作环境已成功从Eclipse迁移到了Android Studio。经过两个项目的使用组员也渐渐适应了。不过你看，只有我对gradle脚本比较熟悉，所以打包的任务当然是我来啦。还好早就搞定了多环境打包，一个命令跑到底，不至于还要改环境打3回。不过跑多了就发现自家的11年mbp已经撑不起这个重任了，不仅卡得严重影响使用，动辄上100度那风扇就像要起飞一样。所以我得回去折磨公司电脑。
<br />
<br />

##### 安装
	安装环境
	Java：JDK 1.8.0_74
	TeamCity：9.1.3
	OS：Ubuntu 15.04

可以从[这里](https://www.jetbrains.com/teamcity/download/)下载最新版TeamCity。

下载后解压，并转到TeamCity根目录，启动
	
	sh bin/runAll.sh start

![Start TeamCity](/assets/images/TeamCity-setup/QQ20160804-1.png)

注意runAll的启动方式启动了平台的主Server和一个位于本机的agent


##### 初始化平台
访问[http://localhost:8111/](http://localhost:8111/)测试平台是否正确启动。每次启动时都会经过一个时间稍长的初始化。

然后会进行一些初次启动设置

![设置配置文件保存路径](/assets/images/TeamCity-setup/QQ20160804-2.png)
设置配置文件保存路径

![设置数据库，本次使用内置数据库](/assets/images/TeamCity-setup/QQ20160804-0.png)
设置数据库，本次使用内置数据库

![会初始化大概5分钟](/assets/images/TeamCity-setup/QQ20160804-3.png)
会初始化大概5分钟

![假装同意条款](/assets/images/TeamCity-setup/QQ20160804-4.png)
假装同意条款

![然后就完成了](/assets/images/TeamCity-setup/QQ20160804-5.png)
然后就完成了


##### 配置工程
点击`Create project from URL`由svn创建工程(也支持git)

填入工程的svn地址及账户等设置，可直接获取svn项目信息创建工程。

工程看上去应该长这样
![Projects home](/assets/images/TeamCity-setup/QQ20160804-9.png)

TeamCity可以识别gradle脚本帮你新建好一个Build Step，所以工程设置里目前应该包含一个Version Control Setting和一个Gradle的Build Step
![Project Setting](/assets/images/TeamCity-setup/QQ20160804-7.png)

注意这个Step的设置中有一项`Gradle Wrapper`(Show advanced options可见)，是否勾选取决于你的工程里有没有包含wrapper(即gradlew文件和gradle/wrapper/..目录)。如果不使用wrapper则需要在agent上配置`GRADLE_HOME`环境变量。总之需要使gradle脚本可以被执行。

进行到这一步就可以点击`Run`来执行编译了。可以查看编译结果、Build log、Commit log等。


##### 自动化
那如何拿到编译好的apk？我们需要设置输出文件。在`General Settings`中找到`Artifact paths`
![Artifact Setting](/assets/images/TeamCity-setup/QQ20160804-8.png)

将apk输出到dist目录下，就可以在首页的Artifacts里看见他们了。(再Run一次才会出现哟)
![Artifacts](/assets/images/TeamCity-setup/QQ20160805-0.png)

我这里将build/apk下的所有文件都输出了，但是这样会将debug版和unaligned包一起输出，如果不想这样可以仅将需要的文件输出，可以配多个，一行一个。

每次都要点Run很麻烦？设置trigger触发自动编译。
![Trigger](/assets/images/TeamCity-setup/QQ20160804-10.png)

常用的trigger有VCS和Schedule，分别可以实现 `当有提交时自动编译` 和 `定时自动编译`。也是可以添多个。不过VCS tirgger是以定时检查实现的，默认每60秒自动check一下svn，并不是严格实时哦。这个时间可以修改，在Administration->Global Settings里。

<br />
这样设置下来，开发只需正常commit，TeamCity自动完成编译。然后测试自己去平台上下载对应环境包测试，省去了发包文件共享的步骤。整个过程完全不需要我参与，真真是解放了生产力。