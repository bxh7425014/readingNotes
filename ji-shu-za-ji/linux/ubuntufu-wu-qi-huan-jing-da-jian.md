ubuntu服务器环境搭建

# 配置Linux环境属性
- 修改终端字体大小
```
sudo dpky-reconfigure console-setup
```
- 第一次root用户解锁操作
  - 修改root密码
  ```
  sudo passwd root
  ```
  - 用户注销
  ```
  logout
  ```
  - ubuntu16.04切换到root用户登录
  ```
  su root
  ```
  - 以root用户进行登录
  - who命令来查看是谁登录。
- 关闭Linux防火墙
  ```
  ufw disable
  ```
  - 卸载iptables组件
  ```
  apt-get remove iptables
  ```
  - 安装vim
  ```
  apt-get install vim
  ```
  - Hostname主机
  ```
  vim /etc/hosts
  ```
# ssh连接配置
- ssh工具：
```
apt-get install openssh-server
```
- 启动ssh服务：
```
/etc/init.d/ssh start
```
- 查看是否服务已经启动：
```
ps -e | grep sshd
```
- 解决默认不能root用户登录使用ssh的问题：
```
vim /etc/ssh/sshd_config
```
随后修改PermitRootLogin，将内容设置为yes
# ftp服务
- 安装
```
apt-get install vsftpd
```
- 修改默认ftp用户的密码
```
passwd ftp
```
ftp完成后，会自动创建目录 /srv/ftp，修改权限
```
chmod 777 /srv/ftp
```
如果要ftp正常工作，修改配置文件
```
vim /etc/vsftpd.conf
```
修改如下参数：
```
anonymous_enable NO //不允许匿名登录
...
local_enable YES //允许本地用户登录
...
write_enable=YES //用户具有写权限
...
chroot_local_user=YES //是否将所有用户限制在主目录
...
chroot_list_enable=YES //是否启动限制用户的名单
```
- 定义名单设置的目录（名单中可以设置多个账号）
```
chroot_list_file=/etc/vsftpd.chroot_list
```
- 增加一个服务配置 pam-service_name=vsftpd ，注意文件中不能存在此条信息的重复数据（我新增了一条，踩了个坑）。
- 增加一个新的文件，写上一个用户
```
vim /etc/vsftpd.chroot_list
```
- 在第一行写上ftp，表示新增一个用户
- 修改 vim /etc/pam.d/vsftpd,注释掉以下内容：
```
#auth required pam_shells.so
```
- 启动服务
```
service vsftpd start
```
# JDK的安装与配置
- 下载Linux版本jdk.
- XShell使用rz命令传到服务器
- 解压
```
root@ubuntu:/home/bianxh/Downloads# tar xzvf jdk-8u111-linux-x64.tar.gz -C /usr/local
```
- 打开环境文件进行配置：
```
vim /etc/profile
```
```
export JAVA_HOME=/usr/local/jdk1.8.0_111
export PATH=$PATH:$JAVA_HOME/bin:
export CLASS_PATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
- 让配置立刻生效
```
source /etc/profile
```
- 测试下是否生效成功
```
root@ubuntu:/usr/local/jdk1.8.0_111# java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
```
# 在Linux中安装Hadoop
- 上传Hadoop文件并解压缩
```
root@ubuntu:/home/bianxh/Downloads# tar xzvf hadoop-2.6.0.tar.gz -C /usr/local
```
- 配置环境变量，修改/etc/profile
```
export HADOOP_HOME=/usr/local/hadoop-2.6.0
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:
```
- 让配置立刻生效
```
source /etc/profile
```
- hadoop依赖jdk
```
root@ubuntu:/usr/local/hadoop-2.6.0/etc/hadoop# vim hadoop-env.sh
```
```
#export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/usr/local/jdk1.8.0_111
```