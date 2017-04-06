Mac Web开发环境配置

\# 安装brew

\`\`\`

/usr/bin/ruby -e "$\(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/master/install\](https://raw.githubusercontent.com/Homebrew/install/master/install\)\)"

\`\`\`

安装下载工具

\`\`\`

brew install wget

\`\`\`

\# 安装Redis

\`\`\`

$ wget [http://download.redis.io/releases/redis-3.0.7.tar.gz](http://download.redis.io/releases/redis-3.0.7.tar.gz)

$ tar xzf redis-3.0.7.tar.gz

$ cd redis-3.0.7

$ make

\`\`\`

启动redis

\`\`\`

$ src/redis-server redis.conf

\`\`\`

连接上以后，关闭redis

\`\`\`

$ src/redis-cli

$ 127.0.0.1:6379&gt; SHUTDOWN

\`\`\`

测试redis

\`\`\`

$ src/redis-cli

redis&gt; set foo bar

OK

redis&gt; get foo

"bar"

\`\`\`

\# Mac OS命令行下使用SublimeText打开文本文件

打开用户配置文件

\`\`\`

vim ~/.bash\_profile

\`\`\`

添加如下alias

\`\`\`

alias subl="'/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl'"

alias ll='ls -al'

\`\`\`

如果不添加别名，也可以选择将路径添加到环境变量下。这里的路径根据实际情况可能会有所不同。

wq保存后回到命令行执行以下命令使其生效：

\`\`\`

source ~/.bash\_profile

\`\`\`

命令行使用，这里我们假设在命令行用SublimeText打开.bash\_profile，则执行如下：

\`\`\`

subl ~/.bash\_profile

\`\`\`

\# 安装邮件服务 James server

\`\`\`

$ subl ~/.bash\_profile

export JAMES\_SRC\_HOME=/usr/local/james-project/james-project

\`\`\`

安装maven，\[参考教程\]\([https://james.apache.org/server/3/dev-build.html\](https://james.apache.org/server/3/dev-build.html\)\)

\`\`\`

$ brew install maven

\`\`\`

默认安装位置：$ /usr/local/Cellar/maven/3.3.9

