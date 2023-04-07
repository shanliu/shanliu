# 进程

1.  后台进程

    ```c
    //当 nochdir为零时，当前目录变为根目录，否则不变；
    //当 noclose为零时，标准输入、标准输出和错误输出重导向为/dev/null，也就是不输出任何信 息，否则照样输出。
    //deamon()调用了fork()，如果fork成功，那么父进程就调用_exit(2)退出，所以看到的错误信息 全部是子进程产生的。如果成功函数返回0，否则返回-1并设置errno。
    //int daemon(int nochdir, int noclose); 
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <fcntl.h>
    #include <limits.h>
    defind PATH_MAX 100
    int main(int argc, char *argv[])
    {
     char strCurPath[PATH_MAX];
     if(daemon(1, 1) < 0)
     {
         perror("error daemon.../n");
         exit(1);
     }
     sleep(10);
     if(getcwd(strCurPath, PATH_MAX) == NULL)
     {
         perror("error getcwd");
         exit(1);
     }
     printf("%s/n", strCurPath);
     return 0;
    }
    ```
2.  daemon 作用类似以下代码

    ```c
    int pid;
    pid=fork();
    if(pid<0)
    exit(1);  //创建错误，退出
    else if(pid>0) //父进程退出
    exit(0);
    setsid(); //使子进程成为组长
    ```
3.  处理子进程结束

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>
    #include <signal.h>
    #include <wait.h>
    void sighf(int sig){
    pid_t t;
    //pid = -1 与 wait 一样
    //pid=0 同组子进程
    //pid>0||pid<-1 指定pid的子进程
    //option WCONTINUED 作业控制 WONHANG 不阻塞返回:0 WUNTRACRD 作业控制未报告返回
    while((t=waitpid(-1,NULL,WNOHANG))>0){
        printf("c:%d\n",t);
    }
    }
    int main(){
    sighandler_t h=signal(SIGCHLD,sighf);
    pid_t tt=fork();
    if(tt==0){
        sleep(20);
        exit(0);
    }
    pause();
    }
    ```
4.  运行程序

    ```c
    //exec函数,返回为子指定程序的exit值
    #include <unistd.h>
    int execl(const char* pathname,const char * arg,.../*(char *)0*/);
    int execv(const char* pathname,char * const argv[]);
    int execle(const char * pathname,const char *arg,...
    /*(char *)0,char * const envp[] */);
    int execve(const char *pathname,char * const argv[],char * const envp[]);
    int execlp(const char * filename,const char *arg,.../*(char *)0 */);
    int execvp(const char * filename,char * const argv[]);
    //execve 内核级调用
    //execlp execvp 从环境变量找执行文件
    //execl execle 指定执行文件 此!!!新的环境变量数组即成为新执行进程的环境变量!!!
    //exe*系列:execve fexecve  execv execle execl execvp execlp
    //第5个字符 l 表示参数列表 v 表示参数数组
    //第6个字符 e 表示指定文件 p 表示从环境中找
    ```

    5.其他

```
#进程标识符
    #include <unistd.h>
    1. pid_t getpid(void); 获取当前进程的ID
    2. pid_t getppid(void); 获取父进程的ID
    3. uid_t getuid(void); 获取进程实际用户ID
    4. uid_t geteuid(void); 获取进程的有效用户ID
    5. gid_t getgid(void); 获取进程的实际组ID
    6. gid_t getegid(void); 获取进程的有效组ID
    7. pid_t fork(void); 创建一个新进程
        创建进程后,两个进程谁先执行是不确定的
        IO是共享的,文件锁不会被继承,其他资源为拷贝
    8. vfork 创建一个新进程
        此函数不拷贝资源副本,且此函数保证子进程先执行

#进程退出 
    exit(0); 正常退出,清理运行环境
    _Exit(); ISO C _exit(); POSIX 函数 不清理运行环境
为正常退出 其他说明为错误退出
    abort(); 产生SIGABRT信号,系统默认处理信号来停止

#父子进程

　　wait 在 SIGCHLD  信号中处理避免阻塞问题
    #include <sys/wait.h>
    父进程比子进程先停止,子进程的父进程将改为init PID:1
    子进程比父进程先停止,子进程僵死Z,需要父进程处理:
    1. pid_t wait (int *statloc); 
        等待任何一个进程退出,进程阻塞
    2. pid_t waitpid(pid_t pid,int *statloc,int option);
         等待指定进程退出
         pid = -1 与 wait 一样 
         pid=0 同组子进程 
         pid>0||pid<-1 指定pid的子进程
         option WCONTINUED 作业控制 WONHANG 不阻塞返回:0 WUNTRACRD 作业控制未报告返回
    处理 wait函数返回
        判断statloc 值
        WIFEXITED(statloc); 子进程正常终止
        WIFSIGNALED(statloc); 子进程异常中止,以下为辅助宏:
            WTERMSIG(statloc); 获取终止子进程的信号编号
            WCOREDUMP(statloc); 判断是否产生core文件(不是所有平台都有)
        WIFSTOPPED(statloc); 子进程处于暂停状态
        WIFCONTINUED(statloc);作业控制继续    

    3. int waitid(idtype_t idtype,id_t id.siginfo *infop,int options);
        类似waitpid 功能
        idtype_t 可选值 P_PID : 指定子进程ID P_PGID :指定组进程子ID P_ALL :何以子进程
        *infop 返回子进程的状态信息
        options 可选值 WCONTINUED WEXITED WONHANG WNOWAIT WSTOPPED
    4. pid_t wait3(int & statloc,int options,struct rusage &rusage);
    5. pid_t wait4(pid_t pid,int *statloc,int options,struct rusage *rusage);

#竞争条件
    由于两个进程谁先执行,所以进程间要相互依赖时候,需要相互等待



#更改用户ID和组ID 无权限修改后值不变
    #include <unistd.h>
    1. int setuid(uid_t uid);
    2. int setgid(gid_t gid); 
    3. int seteuid(uid_t uid); 
    4. int seteuid(uid_t uid); 

    备注:
    root 或chmod u+s 修改实际用户
    root 或chmod g+s 修改实际用户组

#解析器文件:
    #include <stdlib.h>
    int system(const char *cmdstring);
    有调用此函数的程序禁止给root权限

#用户标识:
    #include <unistd.h>
    char * getlogin(void); 出错返回NULL



##进程关系
#进程时间: 子进程清0
    #include <sys/times.h>
    clock_t times(struct tms *buf); 填写当前CPU时间
        struct tms{
            clock_t tms_utime;
            clock_t tms_stime;
            clock_t tms_cutime;
            clock_t tms_cstime;
        };
    clock_t 为滴答数,转换秒处于系统每秒滴答数:sysconf(_SC_CLK_TCK);

#进程组
    与统一作业关联,可以接收来自同一终端的各种信号
    进程组ID默认等于其进程ID
    #include <unistd.h>
    pid_t getpgrp(void); 取得当前进程的进程组ID
    pid_t getpgid(pid_t pid); 获取指定进程的进程组ID 0等于getpgrp
    int setpgid(pid_t pid,pid_t pgid); 设置进程的进程组ID

#会话
    多个进程组 组成一个会话
    #include <unistd.h>
    pid_t setsid(void); 设置一个新会话
    pid_t getsid(pid_t pid); 获得一个会话ID


##信号
    linux:  <bits/signal.h> 
    maxos&&freebsd: <sys/signal.h>
    solaris: <sys/iso/signal_iso.h>
    #include <signal.h>
    子进程将继承父进程的信号处理方法
#UNIX系统信号(具体:**)
    SIGKILL 终止信号
    SIGCHLD 子进程终止向父进程发送的信号
    SIGCLD 系统V的子进程终止信号(不要用)
```
