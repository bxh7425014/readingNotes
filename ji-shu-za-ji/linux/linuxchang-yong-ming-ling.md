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







