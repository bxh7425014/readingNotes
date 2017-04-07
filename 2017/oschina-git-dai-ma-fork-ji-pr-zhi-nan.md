# OSChina git 代码fork及PR指南

## 分支命名规则：

项目代码分为3个特性分支，命名规则为：

开发版本：特性名\\_master

测试版本：特性名\\_test\_branch

正式版本：特性名\\_release\_branch

注：如果是默认项目，特性名可以省略。否则需要加上对应的特性名称，比如当前在当前项目基础上给某单位/组织/地域开发项目时，特性名前缀为对应的拼音，首字母大写，比如给西安开发某项目，分支名称为Xian\_\_\_\_\_\_。

## Fork仓库，本地修改后提交PR

![](http://upload-images.jianshu.io/upload_images/1124873-d77ec58aaf416bbc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Fork之后，个人名下就可以看到Fork出来的仓库了，对应的分支也能看到：

![](http://upload-images.jianshu.io/upload_images/1124873-d52efefad537455f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

找到地址，clone代码下来：

![](http://upload-images.jianshu.io/upload_images/1124873-b3cd82108c9c04a7.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

在默认的master分支，修改Fork出来的本地文件：

![](http://upload-images.jianshu.io/upload_images/1124873-60b2b64f25b104b5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Commit后，push刚才的修改到OSChina个人名下的master分支：

![](http://upload-images.jianshu.io/upload_images/1124873-9c8cdbb0cbd9a123.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

刷新OSChina个人名下仓库，可以看到对应的ReadMe文件已经修改了，点击Pull Request：

![](http://upload-images.jianshu.io/upload_images/1124873-0b5312b0226822fe.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

制定源分支、目标分支、代码审查人员，

![](http://upload-images.jianshu.io/upload_images/1124873-1d91321353a43209.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

PR提交给组织了，上面我们指定了“刘超”审查代码，“卞晓辉”进行测试。可以看到状态是未审核，未测试。

![](http://upload-images.jianshu.io/upload_images/1124873-178e17ff0c71fbc2.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

对应的人员会收到审查和测试通知，如下：  
![](http://upload-images.jianshu.io/upload_images/1124873-822dcfda7f80e1e1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

审查和测试通过后，可以接受代码合并了，如下：

![](http://upload-images.jianshu.io/upload_images/1124873-3d702887b2c41055.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

合并后，如果还有一次反悔的机会，慎重使用：

![](http://upload-images.jianshu.io/upload_images/1124873-205ea7f897dbb4c1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

可以看到对应的修改已经更新到主仓库上（XAVito/HelloWorld）。如下：

![](http://upload-images.jianshu.io/upload_images/1124873-87bb165a503e5d57.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Fork仓库同步主仓库的最新代码

主仓库主分支更新了内容，如下：

![](http://upload-images.jianshu.io/upload_images/1124873-b0ec26779d130435.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Fork仓库同步主仓库的更新：

![](http://upload-images.jianshu.io/upload_images/1124873-12970863eec6e68d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

更新后查看差异（注意当前在Fork仓库的master分支上），有如下两种方式：

![](http://upload-images.jianshu.io/upload_images/1124873-cda26b8a2e96b307.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-09c8ff0e6a879523.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-8145c10ac70fef06.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

测试在Fork出来的仓库master分支做增量修改并提交（预期会产生冲突）。如下：

![](http://upload-images.jianshu.io/upload_images/1124873-cbcb2ee21cbfd0c1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-691c4f54038cdd90.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-0e1be28c0bbb3cbc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

试用merge，产生冲突，如下：

![](http://upload-images.jianshu.io/upload_images/1124873-6362cb1c16e24548.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-c5ff4cbe47c28501.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

解决冲突，并提交代码到Fork仓库：

![](http://upload-images.jianshu.io/upload_images/1124873-5d1721ded279f9fc.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-b72fb7d848a2f017.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

但与此同时主仓库也前进了，如下：

![](http://upload-images.jianshu.io/upload_images/1124873-8a9e99e74394ba73.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-e24c937f62990392.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-5e32805be1db5015.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-996188a474b64044.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Fork仓库提交更新到主仓库（PR审查代码勾选上），提交PR后，主仓库收到更新通知了：![](http://upload-images.jianshu.io/upload_images/1124873-51f6b226fb43598c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

审查没有问题，通过，发现和主仓库的修改有冲突了，如下：![](http://upload-images.jianshu.io/upload_images/1124873-85626684d12696fe.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

好办，解决冲突：

![](http://upload-images.jianshu.io/upload_images/1124873-bbcd66a3f74fb6e5.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1124873-56be3c50955ce64e.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)![](http://upload-images.jianshu.io/upload_images/1124873-bed54a5cd6196fbe.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

注：对于冲突的代码，代码审核人员必须要大家解决冲突，对于存在冲突的PR，关闭PR并评论。

进一步学习

合并最近的多次提交

[参考地址](https://git-scm.com/book/zh/v2/Git-工具-重写历史)

变基

分支是很让人讨厌的，因为代码提交历史不够整洁。也有办法让他归到一个分支上去，使用rebase。方法请参考【[变基](https://git-scm.com/book/zh/v2/Git-分支-变基)[】。](https://git-scm.com/book/zh/v2/Git-分支-变基%29】。)

