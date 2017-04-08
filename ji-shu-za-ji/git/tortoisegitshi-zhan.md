2016-07-13 TortoiseGit实战.md
[TOC]
很多童鞋都不习惯使用git命令。这里就使用TortoiseGit进行git的常用操作。
# 码云上创建测试项目
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-239bbae0cea6c61f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
创建后的项目代码地址是：
git@git.oschina.net:yyxz/yyxz-Test.git
# 克隆项目代码到本地
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-8fa75611f9d7b202.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 为了方便后续演示代码冲突解决情况，也克隆一个备份到本地D:\Workspaces\git\yyxz-Test\Test2\yyxz-Test
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-9aedf65680c43d1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，默认的分支跟踪关系如下：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-6b1e8979fed43cc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 解决冲突
编辑文本，并提交：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-3d456cac1e01a740.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-d92b55db3ccdf491.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-3b52aa1ab8619ce4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-a54f307a19bd4de0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
在Test2目录添加测试代码：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-c51c99c6e36b9e0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
commit成功，但是push后又冲突，如下：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-d6d64f1febeaf135.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
展开pull按钮旁边的三角箭头，选择fetch，点击OK。
>使用 git status命令，可以随时查看本地仓库状态，比如现在有冲突。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-46a0fbdb45ae0d9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
右键菜单，TortoiseGit-->switch checkout 到master分支，然后选择merge
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-7c895295ea85c49a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到冲突了：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-fd1efa765e30a2db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-e593d83017f8c07d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-51d627fbf9696e3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
右击冲突文件，commit，
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-2b300087d405c954.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Edit Confilicts，
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-da0bdc8001903eca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
编辑完冲突后，右击冲突文件，选择Resolved。然后commit，push即可。
在Test1目录进行更新，可以看到更新了。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-6a33bf5c813ebe06.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-4660c06140b7d85e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#创建分支
右键菜单，TortoiseGit --> Create Branch，
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-5ee3abb68dc18456.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
查看分支跟踪关系，右键菜单，TortoiseGit --> Browse Refrences，可以看到分支跟踪关系，如下：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-abb0a1151fee4d8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以右击test，设置跟踪分支Select Tracked Branch。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-cff2ad74c0939542.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
切换到test分支，添加测试代码：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-6609bc26ed07adf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-0fdef538e12841ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以从分支图看到test分支有新的演化：
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-b79bbe54f75dd7c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
将test分支的修改合并merge到master分支：
先切换到master分支，然后merge from test分支。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-618af569729a8d68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到merge成功了。同时我们也可以看到之前的冲突合并历史。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-3eed45178d7dd8d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
最后，删除本地test分支。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-19300f8cee239431.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当然，也可以删除远程test分支。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-4f31fbd2ca3b0797.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，相比之前，test分支没有了（右键菜单，Revision Graph可以看到）
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-9d3f314fab858822.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![一一小知](http://upload-images.jianshu.io/upload_images/1124873-f73f5515b7174f8d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)