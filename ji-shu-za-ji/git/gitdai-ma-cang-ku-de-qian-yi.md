git代码仓库如何迁移

从原地址克隆一份裸版本库。
语法：
```
git clone --bare <repository> <directory.git>
```
例如：
```
 git clone --bare git@git.oschina.net:yiyixiaozhi/BookCode.git
```

然后到新的 Git 服务器上创建一个目标项目（准备迁移过去）。比如新项目地址是：
```
git@github.com:bxh7425014/BookCode.git
```

以镜像推送的方式上传代码到 GitCafe 服务器上，如下：
```
$ cd BookCode.git/
$ git push --mirror git@github.com:bxh7425014/BookCode.git
```