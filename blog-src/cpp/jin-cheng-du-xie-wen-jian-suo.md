# 进程读写文件锁

> 来源PHP源码

```c
#  define FCGI_LOCK(fd)                                \
    do {                                            \
        struct flock lock;                            \
        lock.l_type = F_WRLCK;                        \
        lock.l_start = 0;                            \
        lock.l_whence = SEEK_SET;                    \
        lock.l_len = 0;                                \
        if (fcntl(fd, F_SETLKW, &lock) != -1) {        \
            break;                                    \
        } else if (errno != EINTR || in_shutdown) {    \
            return -1;                                \
        }                                            \
    } while (1)

#  define FCGI_UNLOCK(fd)                            \
    do {                                            \
        int orig_errno = errno;                        \
        while (1) {                                    \
            struct flock lock;                        \
            lock.l_type = F_UNLCK;                    \
            lock.l_start = 0;                        \
            lock.l_whence = SEEK_SET;                \
            lock.l_len = 0;                            \
            if (fcntl(fd, F_SETLK, &lock) != -1) {    \
                break;                                \
            } else if (errno != EINTR) {            \
                return -1;                            \
            }                                        \
        }                                            \
        errno = orig_errno;                            \
    } while (0)
```

备注 error == EINTR 为中断错误.如信号 使用 fd 可以是文件.socket连接等

```c
int fd=open("conftest_in", O_WRONLY|O_CREAT, 0600);
FCGI_LOCK(fd);
//file 读写操作...                                        FCGI_UNLOCK(fd);
close(fd);
```
