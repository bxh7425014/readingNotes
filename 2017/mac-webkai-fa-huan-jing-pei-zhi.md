Mac Web开发环境配置

# 安装brew

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install\"
```

安装下载工具

```
brew install wget
```

# 安装Redis

```
$ wget http://download.redis.io/releases/redis-3.0.7.tar.gz
$ tar xzf redis-3.0.7.tar.gz
$ cd redis-3.0.7
$ make
```

启动redis

```
$ src/redis-server redis.conf
```

连接上以后，关闭redis

```
$ src/redis-cli
$ 127.0.0.1:6379> SHUTDOWN
```

测试redis

```bash
$ src/redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```

\# Mac OS命令行下使用SublimeText打开文本文件

打开用户配置文件

```
vim ~/.bash_profile
```

添加如下alias

```
alias subl="'/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl'"
alias ll='ls -al'
```

如果不添加别名，也可以选择将路径添加到环境变量下。这里的路径根据实际情况可能会有所不同。

wq保存后回到命令行执行以下命令使其生效：

```
source ~/.bash_profile
```

命令行使用，这里我们假设在命令行用SublimeText打开.bash\_profile，则执行如下：

```
subl ~/.bash_profile
```

# 安装邮件服务 James server

```
$ subl ~/.bash_profile
export JAMES_SRC_HOME=/usr/local/james-project/james-project
```

安装maven，[参考教程](https://james.apache.org/server/3/dev-build.html)

```
$ brew install maven
```

默认安装位置是

```
$ /usr/local/Cellar/maven/3.3.9
```

# Fix problems

maven报错：

```
Could not calculate build plan: Plugin org.apache.maven.plugins:maven-resources-plugin:2.6 or one of its dependencies could not be resolved: Failed to read artifact descriptor for org.apache.maven.plugins:maven-resources-plugin:jar:2.6
```

解决方法：

1、查看windows -&gt; Preferences -&gt; maven 的settings.xml文件中.m2的位置

2、然后将.m2/repository/org/apache/maven/plugins目录下的文件夹全部删除

3、选中maven项目，右键 --&gt; maven --&gt; update，让maven重新下载依赖包

> 注：此IDE自带maven插件，不需要再自己下载安装maven插件



