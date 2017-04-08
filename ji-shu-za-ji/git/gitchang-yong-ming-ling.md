git常用命令
#git 命令
文件的状态变化周期
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-c13c1ffd98f93353.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
检查当前文件状态
$ git status

跟踪新文件
$ git add README //READ ME进入已暂存状态

状态简览
$ git status -s
 M README
MM Rakefile 
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt

忽略文件
.gitignore

$ cat .gitignore
*.[oa] //忽略所有以 .o 或 .a 结尾的文件
*~  //忽略所有以波浪符（~）结尾的文件

查看已暂存和未暂存的修改
git diff
git diff --cached //查看已暂存的将要添加到下次提交里的内容

提交更新
git commit -m "Story 182: Fix benchmarks for speed"
git commit -am "Story 182: Fix benchmarks for speed" //跳过使用暂存区域

从已跟踪文件清单中移除,从暂存区域移除
下一次提交时，该文件就不再纳入版本管理了.
$ git rm PROJECTS.md
$ git rm --cached README //从暂存区域移除），但保留在当前工作目录中

移动(重命名)文件
$ git mv file_from file_to

查看提交历史
$ git log

取消暂存的文件
$ git reset HEAD CONTRIBUTING.md

撤消对文件的修改
$ git checkout -- CONTRIBUTING.md

查看远程仓库
$ git remote -v

添加远程仓库
git remote add pb https://github.com/paulboone/ticgit

从远程仓库中抓取与拉取
git fetch 命令会将数据拉取到你的本地仓库 - 它并不会自动合并或修改你当前的工作。 当准备好时你必须手动将其合并入你的工作。
git pull 通常会从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支。

推送到远程仓库
git push [remote-name] [branch-name]
```

#git 分支
```
分支创建
$ git branch testing  //创建分支
$ git checkout -b iss53 //新建一个分支并同时切换到那个分支上
$ git checkout master //切换分支

//合并hotfix到master分支， git commit 来完成合并提交
$ git checkout master
$ git merge hotfix

//删除分支
git branch -d iss53

//查看每一个分支的最后一次提交
$ git branch -v

//新建本地分支，推送到远程，然后设置跟踪关系
$ git branch testing2
$ git push origin testing2:testing2-remote
$ git checkout testing2
$ git branch --set-upstream-to=origin/testing2-remote

//本地新建一个远程同名分支并跟踪
$ git checkout -t origin/testing-remote
```


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1124873-9c2bbaee4af96000.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)