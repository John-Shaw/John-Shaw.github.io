---

layout: post
title: 在Xcode中使用Github
categories:
- 利器
tags:
- Xcode
- Github

---
现代IDE普遍集成了版本控制工具(Git,SVN,etc)，比如脑浆喷射([jetBrain](http://jetbrains.com/))公司的招牌产品IntelliJ IDEA，可以轻松连接Github账号，对项目进行版本管理。Xcode虽然也支持Git，但却并不如IDEA那么傻瓜一键。我在网上搜索很多教程都是通过git命令来操作，但实际上完成这个目标并不需要敲任何shell命令。下面我们来介绍如何在Xcode中使用Github。
********

1. 在Github新建一个repo
--------------------------

Xcode并不能直接连接Github账号，所以我们必须在Github上新建一个repo。

![创建Github上的repo](http://john-shaw.github.io/media/pic/2015/04/xcode-github-1.png =618x446)

值得注意的是在add .gitignore选项中选择Objective-C，这样会使用Github的OC项目忽略模版，对于像我这样的新手而言，省了很多配置的功夫。
2. 在Xcode中新建一个工程
-------------------------
这里不多说，记得勾选Source Control选项，他会自动创建一个本地的git repo。

![创建Xcode项目](http://john-shaw.github.io/media/pic/2015/04/xcode-github-2.png)
3. 为你的本地repo添加一个remote
-------------------------
选择Source Control -> [your repo name] -> Configure [your repo name]

![添加remote－1](http://john-shaw.github.io/media/pic/2015/04/xcode-github-3.png)

然后在remote选项卡下，点"+"号添加一个remote即可(我这里已经添加好了)。name可以随便填，address填写你的github repo地址。例如 *https://github.com/[your name]/[your repo name]***.git**(注意url后面加上.git)

![添加remote－2](http://john-shaw.github.io/media/pic/2015/04/xcode-github-4.png)

**OK!**

现在你只需要分别pull和push一次，将Github repo的.gitignore,README.md等拉取本地，再push本地项目文件至Github(可能需要先commit)即可。具体Git操作不在赘述。

可以开工写代码啦。(●'◡'●)

