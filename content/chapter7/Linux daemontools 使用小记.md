# Linux daemontools 使用小记

daemontools的安装和使用可参考[进程的守护神 - daemontools](<http://linbo.github.io/2013/02/24/daemontools>)[^1]，目前（2019.05）daemontools已经许久未更新，停留在[0.76](http://cr.yp.to/daemontools/daemontools-0.76.tar.gz)。

#### 安装时如果遇到错误

> /usr/bin/ld: errno: TLS definition in /lib64/libc.so.6 section .tbss mismatches non-TLS reference in envdir.o

则编辑src/conf-cc，在gcc行末尾加上`-include /usr/include/errno.h` 重新安装即可。

#### run脚本注意事项

> 关于run脚本：切记，要用 exec 执行最终执行服务的程序，否则运行 run 脚本的shell收到 svc -d 的 TERM 信号退出之后，实际执行服务的那个程序不会跟着退出[^2]。

还需要注意的是，在执行命令时，一定不能通过`&`后台方式执行，否则无法监控真实的运行状态：pid不断地更新，启动时间一直在0-1之间。

#### 不断重启参考资料
1. 命令错误[^2]

    因为supervise是通过监控run退出时产生的SIGCHLD信号来识别服务已经终止，并重启服务的。
    如果这里没exec，则会导致fork+exec效果，在svc -d终止服务时，只给run脚本发送TERM命令，而run脚本fork出来的子进程不会收到信号，从而变成孤儿进程继续运行，占据文件锁、TCP端口等资源。
    对于不便exec的程序，可以在后面加&符号后台运行，并在调用命令之后用 waitpid %1 命令等待，从而阻止run脚本退出；run脚本开头处应该用trap命令捕获TERM信号，信号处理过程中给%1发送TERM信号，即可实现整体退出的效果。


2. 服务错误[^3]
    Sometimes however daemon tools will report the service is up but it really isn't as it is trying to continually start the process and it's failing every time. In that case daemontools will say it's up but each time you run the svstat command it will show a new pid and the uptime will have been reset.

    ```shell
    [root@admin:~] cd /var/service
    [root@admin:/service] svstat memcached
    memcached: up (pid 1842) 1 seconds
    [root@admin:/service] svstat memcached
    memcached: up (pid 1854) 1 seconds
    [root@admin:/service] svstat memcached
    memcached: up (pid 1859) 0 seconds
    ```
    
    In that case you can try to figure out why the process is not starting by examining the logs or look for a "readproctitle" error message. readproctitle is a process that is part of daemontools that shows the last 400 bytes of error messages that are produced by the svscan process. You can check it by running a command like:
    ```shell
    ps -Af | grep readproctitle
    ```

[^1]:http://linbo.github.io/2013/02/24/daemontools
[^2]:http://naixwf.github.io/2015/07/21/2014-11-19-daemontools
[^3]:https://www.ianlewis.org/en/using-daemontools