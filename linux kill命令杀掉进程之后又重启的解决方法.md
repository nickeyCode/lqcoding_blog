# linux kill命令杀掉进程之后又重启的解决方法

今天博客突然进不去了，进入发现cpu使用率100%，好家伙。。。。

`top`发现一个进程`perfctl`占用198%

尝试kill -9 发现kill不掉

这情况一般来说是有父进程没杀掉

首先获取到进程的`pid`

通过`cp /proc/${PID}`进入到目录

cat status 找到父进程PPID。

kill -9 PPID  kill掉父进程



到这里就已经让cpu使用率正常了，但是下一步要查看下这是如何被入侵