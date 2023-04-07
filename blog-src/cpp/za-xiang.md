# 杂项

1.  判断WIN还是LINUX

    ```c
    #if defined(WIN32) || defined(WIN64)
    #include <windows.h>
    #define sleep(n) Sleep(1000 * (n))
    #else
    #include <unistd.h>
    #endif
    ```
2.  ```c
    int *a=calloc(3,sizeof(int));//分配连续内存空间
    int *a=malloc(sizeof(int));//分配指定大小内存空间
    int *a=realloc(a,sizeof(int)*3);//调整内存空间,一般大调小使用,小调大会先释放原内存重新申请
    free(a);//释放内存;
    exit;//正常停止
    abort();//异常停止
    int a=rand();//返回一个随机整数
    char *a=getenv("PATH");//得到一个环境变量
    putenv("VAR=VALUE");//增加环境变量
    //数字字符操作 start
    int a=labs(-10);//绝对值
    //以下3个非标准C函数
    float a=atof("10.1");//字符串转浮点
    int a=atoi("10");//字符串转整形
    long int a=atol("10");//字符串转整形
    //浮点转字符double value= 86.789e5;int dec, sign;int ndig = 5;char *p = ecvt(value,ndig,&dec,&sign);//参数:浮点数,要转换的位数,小数点的位置(返回),正或负数(返回)//返回:连续的字符串,无小数点char *p = fcvt(value,ndig,&dec,&sign);//跟上面一样,但结果会四舍五入
    //整形转字符
    string[10];
    itoa(int, string*, 10);//整数,存储的字符串,整数进制(2,10等) 后面没有\0结束
    ```

    ```
    lseek (int __fd, __off_t __offset, int __whence) ;//偏移
    alarm (unsigned int __seconds);//闹钟函数，它可以在进程中设置一个定时器，当定时器指定的时间到时，它向进程发送SIGALRM信号
    sleep (unsigned int __seconds);//秒暂停
    usleep (__useconds_t __useconds);//毫秒暂停
    pause (void);//暂停
    chown (__const char *__file, __uid_t __owner, __gid_t __group);//更改文件权限chdir (__const char *__path);//改根目录
    dup (int __fd); or dup2 (int __fd, int __fd2);//复制文件描述符
    sysconf (int __name);//得到系统信息
    isatty (int __fd);
    link (__const char *__from, __const char *__to);
    unlink (__const char *__name); //删文件
    rmdir (__const char *__path);//删目录
    gethostname (char *__name, size_t __len);
    chroot (__const char *__path);
    daemon (int __nochdir, int __noclose);
    getpagesize (void);//页大小
    getdtablesize (void);
    sync (void);//缓存写入系统
    ```
