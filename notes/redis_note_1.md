又挖坑了

看4.0版本

整体架构很多书，博客都说了。简单列出几个看到的有意思的东西

main在server.c中 ~~3.0版本是redis.c~~

1.  _redisPanic函数有一段

```
void _serverPanic(const char *file, int line, const char *msg, ...) {
...log...
#ifdef HAVE_BACKTRACE
    serverLog(LL_WARNING,"(forcing SIGSEGV in order to print the stack trace)");
#endif
    serverLog(LL_WARNING,"------------------------------------------------");
    *((char*)-1) = 'x';
}
```

指针访问-1  故意段错误，[这里也有讨论](https://stackoverflow.com/questions/20844863/what-does-char-1-x-code-mean)

2 变成守护进程 大家实现都一样，这几行都能背下来了

```
void daemonize(void) {
    int fd;

    if (fork() != 0) exit(0); /* parent exits */
    setsid(); /* create a new session */

    if ((fd = open("/dev/null", O_RDWR, 0)) != -1) {
        dup2(fd, STDIN_FILENO);
        dup2(fd, STDOUT_FILENO);
        dup2(fd, STDERR_FILENO);
        if (fd > STDERR_FILENO) close(fd);
    }
}
```

  

3 查Linux支不支持overcommit_memory

    int linuxOvercommitMemoryValue(void) {
        FILE *fp = fopen("/proc/sys/vm/overcommit_memory","r");
        char buf[64];
    if (!fp) return -1;
    if (fgets(buf,64,fp) == NULL) {
        fclose(fp);
        return -1;
    }
    fclose(fp);
    
    return atoi(buf);
    }
关于这个参数，见<http://linuxperf.com/?p=102>

> 如果/proc/sys/vm/overcommit_memory被设置为0，并且配置了rdb重新功能，如果内存不足，则frok的时候会失败，如果在往redis中塞数据，
> 会失败，打印 MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk
> 如果/proc/sys/vm/overcommit_memory被设置为1，则不管内存够不够都会fork失败，这样会引发OOM，最终redis实例会被杀掉。



4 setlocale(LC_COLLATE,"");

local这东西概念特别多。为什么用LC_COLLATE？搜了一圈也没有啥解释，可能这种比较场景就用它吧。



5 pid file 

`void createPidFile(void) {`
​    `/* Try to write the pid file in a best-effort way. */`
​    `FILE *fp = fopen(server.pidfile,"w");`
​    `if (fp) {`
​        `fprintf(fp,"%d\n",(int)getpid());`
​        `fclose(fp);`
​    `}`
`}`

守护进程都有的操作。

参考https://stackoverflow.com/questions/8296170/what-is-a-pid-file-and-what-does-it-contain#

https://unix.stackexchange.com/questions/12815/what-are-pid-and-lock-files-for



6 非阻塞fd  anetNonBlock(char *err, int fd ）

    if ((flags = fcntl(fd, F_GETFL)) == -1) 
    if (fcntl(fd, F_SETFL, flags | O_NONBLOCK) == -1)
    就是这个
    ::fcntl(sock, F_SETFL, O_NONBLOCK | O_RDWR);
    
也是基本操作了。



7 initServer 

主要逻辑 asCreateFileEvent aeCreateTimeEvent 注册handle

最后有后台线程任务初始化bioinit，主要功能在bioProcessBackgroundJobs，消费队列。

