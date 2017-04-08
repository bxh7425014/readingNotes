Linux常用命令

- 设置当前系统时间命令示例：
```
$ date –set “10/15/2009 20:18”
```

- 递归解压bz2g格式压缩包示例：

```
tar -jxvf cygwin.tar.bz2 -C 指定目录
```

- Samba服务器文件夹权限分配实例（修改/etc/samba/smb.conf）：

```
[bianxh]
  comment = File Share of bianxh
  browseable = yes
  path = /home/bianxh/
  writable = yes
  create mask = 777
  valid users = public bianxh
  directory mask = 777
  available = yes
```

#Git
-repo安装（管理员权限）
1、拷贝repo文件至/usr/bin目录下
2、增加+x权限即可：sudo chmod +x /usr/bin/repo
更新.repo分支信息命令：
进入manifests文件夹，
git remote update
git checkout origin/master
- 我们经常使用的几个git命令：
~~~
git checkout -t origin/分支名 创建跟踪分支
git checkout -b 分支名 创建本地分支
git branch -a 查看本git库的所有分支
git branch 查看自己创建的分支
git status 查看当前修改了哪些文件
git diff 查看当前修改内容
git commit -m "注释" 提交修改到当前分支
git push 上传到服务器
git pull 更新本库及本地代码
git cherry-pick 提交号 将其它分支上的提交挪到当前分支上
repo sync . 作用同git pull
~~~
- 回退本地的提交示例：
git reset --hard 8963bbd4004a60630b1aa062cbb2822d97ced799
- 查看远程仓库的log示例：
git log remotes/origin/branch_xxx





