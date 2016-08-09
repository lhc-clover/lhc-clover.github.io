---
layout: post
title:  "使用Nexus搭建私有仓库，代理JCenter"
date:   2016-03-29 11:15:00
category: Tools
---
<br/>
由于大家都懂的不能明说的原因，国内访问许多仓库都非常痛苦。Gradle sync一跑就是一小时不知大家遇到过没。尤其是引入一个库后其他两位同事还要重新痛苦一次。Nexus就是将痛苦x3变为痛苦x1的神器啦。
<br/>
<br/>

##### 配置

Nexus需要Java，过程略    
[下载](http://www.sonatype.com/download-oss-sonatype)并解压Nexus Repository OSS

修改`conf/nexus.properties`配置文件更改默认端口防止冲突    
修改`bin/nexus`打开`RUN_AS_USER=root`不然无法启动    
	
	打开后启动时会提示`NOT RECOMMENDED TO RUN AS ROOT`，但不打开我这里确实是起不起来的

启动`.bin/nexus start`    
查看状态`.bin/nexus status`	

##### 仓库管理

Android库大多使用JCenter，所以第一步应在私有仓库上设置一个JCenter镜像

![Add proxy repository](/assets/images/nexus-repository-oss-setup/1.png)
<center>新增一个proxy类型的仓库</center>
<br/>
![Setup repository](/assets/images/nexus-repository-oss-setup/2.png)
<center>配置仓库设置，使其指向jcenter的中央仓库地址，注意打开Download Remote Indexes</center>
<br/>
![Add to public](/assets/images/nexus-repository-oss-setup/3.png)
<center>将新建的jcenter代理仓库加入public中</center>
<br/>
大名鼎鼎的jitpack也放进去，同样处理

![Jitpack settings](/assets/images/nexus-repository-oss-setup/4.png)
<center>Jitpack设置</center>
<br/>
将需要的仓库都设置完成后，下一步是修改工程的gradle配置，替换原来的jcenter使其指向私有仓库地址

![Jitpack settings](/assets/images/nexus-repository-oss-setup/5.png)
<center>build.gradle配置</center>
<br/>
这样Nexus会自动帮咱们代理工程中请求过的包，第二次以后就可以不必再经历恼人的sync了。

##### 其他

当然，私有仓库的玩法还有很多。例如使用`3rd party`仓库发布公司内部使用的包，甚至正在开发需要频繁修改的包也可以用`Snapshots`仓库解决。不过这些都是另外的故事了...