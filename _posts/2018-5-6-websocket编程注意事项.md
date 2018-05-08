[Linux下高并发socket最大连接数所受的各种限制](https://blog.csdn.net/guowake/article/details/6615728)

最高的并发数量都要受到系统对用户单一进程同时可打开文件数量的限制

1. 命令`ulimit -n`
查看系统允许当前用户进程打开的文件数限制

操作系统提供**限制可使用资源量**的方式
```
max_conn => 10000, 此参数用来设置Server最大允许维持多少个tcp连接。超过此数量后，新进入的连接将被拒绝。

此参数不要调整的过大，根据机器内存的实际情况来设置。Swoole会根据此数值一次性分配一块大内存来保存Connection信息

ulimit -n
    1024000
ulimit -a
    core file size          (blocks, -c) 0
    data seg size           (kbytes, -d) unlimited
    scheduling priority             (-e) 0
    file size               (blocks, -f) unlimited
    pending signals                 (-i) 18593
    max locked memory       (kbytes, -l) 64
    max memory size         (kbytes, -m) unlimited
    open files                      (-n) 1024000
    pipe size            (512 bytes, -p) 8
    POSIX message queues     (bytes, -q) 819200
    real-time priority              (-r) 0
    stack size              (kbytes, -s) 8192
    cpu time               (seconds, -t) unlimited
    max user processes              (-u) 1024000
    virtual memory          (kbytes, -v) unlimited
    file locks                      (-x) unlimited



cat /proc/sys/fs/file-nr
3200	0	19786373
1.已经分配的文件句柄数，2.已经分配但没有使用的文件句柄数，3.最大文件句柄数。
```