# 信号

## 信号

```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
#include <setjmp.h>
static sigjmp_buf jmpbuf;
void sighandle(int sig){
    printf("hi\n");
    fflush(stdout);
        siglongjmp(jmpbuf,1);
}
int main(){
    if(signal(SIGUSR1, sighandle) == SIG_ERR){
            perror("signal");
            return -1;
    }
    printf("start main\n");
    if(sigsetjmp(jmpbuf,1)){ //1 将移除信号屏蔽 0 不移除
            printf("return from sighandle\n");
    }
    while(1){
            pause();
    }
}
```

```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>
#include <setjmp.h>
void sighandle(int sig){
    printf("hi\n");
    fflush(stdout);
}
void sighandle1(int sig,siginfo_t *info,void *reverse)
{
    printf("hi1\n");
    fflush(stdout);
}
//通过mask 信号掩码可以控制信号的先后接受顺序
int main(){
    //SA_INTERRUPT 信号中断的系统调用不会自动重启动
    //SA_RESTART 信号中断的系统调用会自动重启,即信号处理完成时回到中断时的系统调用中
    //SA_NOCLDSTOP 当子进程停止时不产生此信号。当子进程终止时才产生此信号
    //SA_NOCLDWAIT 当调用进程的子进程终止时不创建僵尸进程 父进程在后调用wait 则阻塞，直到所有子进程都终止
    //SA_SIGINFO 可以得到产生此信号发生时的相关信息 siginfo_t
    struct sigaction sa_usr;
    sa_usr.sa_flags = SA_INTERRUPT;
    sa_usr.sa_handler = sighandle;   //信号处理函数
    sigemptyset(&sa_usr.sa_mask);
    sigaction(SIGINT, &sa_usr, NULL);

     struct sigaction sa_usr1;
    sa_usr1.sa_flags = SA_RESTART|SA_SIGINFO;
    sa_usr1.sa_sigaction = sighandle1;   //信号处理函数
    sigemptyset(&sa_usr1.sa_mask);
    sigaction(SIGUSR1, &sa_usr1, NULL);

    printf("start main\n");
    char b[100];
    scanf("%s",b);
}
```

### 信号

```
linux:  <bits/signal.h> 
maxos&&freebsd: <sys/signal.h>
solaris: <sys/iso/signal_iso.h>
#include <signal.h>
子进程将继承父进程的信号处理方法
```

## UNIX系统信号(具体:\*\*)

```
SIGKILL 终止信号
SIGCHLD 子进程终止向父进程发送的信号
SIGCLD 系统V的子进程终止信号(不要用)
```

## signal 函数

1. void (\_signal(int signo,void(\_func)(int)))(int);
2. typeof void sigfun(int);\
   sigfun _signal(int,sigfun_ );

## 系统信号处理函数(宏)

```
SIG_ERR
SIG_DFL 
SIG_IGN
```

## 可重入函数

```
因为无法预知程序执行情况,所以在实现信号处理函数时候,不要使用不可重入函数
```

## 可信信号术语

```
重复多个相同信号(未处理的时候)只保留传送一个,信号传送没有顺序
```

## 发送信号

```
#include <signal.h>
int kill (pid_t pid,int signo); 向指定进程发送信号,需要权限
    pid >0 发送信号到指定进程
    pid==0 发送给自己
    pid ==-1 给所有有权限发送信号的进程发送信号
    pid <0 pid的绝对值发送信号
int raise(int signo); 给自己发送信号

#include <unistd.h>
unsigned int alarm(unsigned int seconds); 设置计时器
    但上一个计时器没完成,在调用者复位定时器重新计时
int pause(void); 产生一个暂停信号
```

## 信号集 作用,告诉内核不可以发生该信号集的信号

```
#include <signal.h>
int sigemptyset(sigset_t *set); 清除所有信号
int sigfillset(sigset_t *set); 重置信号
int sigaddset(sigset_t *set,int signo); 添加信号
int sigdelset(sigset_t *set,int signo); 删除信号
int sigismember(const sigset_t *set,int signo); 检查是否有指定信号

int sigprocmask(int how,const sigset_t *restric set,sigset_t *restirc oset);
    把建立好的信号提交给内核或从内核取出当前的屏蔽信号
    oset 非空,当前进程的屏蔽信号通过oset返回
    set 非空,根据how的指示来修改当前的屏蔽信号 (对于SIGKILL SIGSTOP无效)
    how 取值: SIG_BLOCK 新加新的屏蔽信号 (并集)
              SIG_UNBLOCK 删除指定的屏蔽信号 (交集)
              SIG_SETMASK 用当前屏蔽信号代替原来的屏蔽信号
int sigpending(sigset_t *set);
    当信号被屏蔽之后,只是不传送给程序,信号在内核阻塞
    调用此程序可以获取被阻塞的这些没有处理的信号
    当屏蔽取出之后,信号将立即会被传送,相当于调用sigprocmask解除屏蔽后执行
```

## 新信号处理函数

```
#include <signal.h>
int sigaction(int signo,const struct sigaction *restrict act,
              struct sigaction * restrict oact);
    信号处理函数设置,可以代替signal函数
    signo 信号编号
    act 非空,修改信号处理动作
    oact 非空.返回原来的处理动作
    struct sigaction{
        void (*sa_handler)(int);/*信号处理函数*/
        sigset_t sa_mask; 
            /*发生信号处理时,将该信号添加到屏蔽信号集,必须初始化0
            返回时候恢复原信号集(为了保证未返回时候正常,需要使用siglongjmp函数),
            可以让某些信号在此信号之后触发*/
        int sa_flags;
            /*对信号处理的选项,对信号的发生的调整,
            可以多选值,用 | 连接参考page 262,表10-5*/
        void (*sa_sigaction)(int,siginfo_t *,void *);
        /*但sa_flags 为SA_SIGINFO时候用该函数处理信号
          用此函数处理信号可以得到产生此信号发生时的相关信息
          关于siginfo_t 的结构可以参考page 263*/
    }
```

## sigsetjmp与siglogjmp 函数

```
但信号进入信号处理函数前,当前信号会被添加到信号屏蔽集里
但时候logjmp跳出该信号处理函数时候,有些系统不会将当前信号从信号屏蔽里移除
所以为了兼容,在信号处理函数里面,使用siglogjmp跳转程序
siglogjmp 还会恢复sigsetjmp保留的关键子
#include <setjmp.h>
int sigsetjmp(sigjmp_buf env,int savemask); 设置一个跳转点
void siglongjmp(sigjmp_buf env,int val); 跳转到指定跳转点
```

```php
#include<stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
#include <setjmp.h>
static jmp_buf jmpbuf;
void sig_handler(int num)
{
    printf("\nrecvive the signal is %d\n", num);
    siglongjmp(jmpbuf, 2);//
}
int main()
{
    int time = 5;
    signal(SIGINT, sig_handler);
    //若savemask为非0值，则sigsetjmp在env中保存进程的当前屏蔽字,不屏蔽信号会存在多次触发问题
    //默认调用siglongjmp设置,第一次执行以下都返回0
    printf("1:%d",sigsetjmp(jmpbuf, 1/*savemask*/));
    printf("2:%d",sigsetjmp(jmpbuf, 1/*savemask*/));
    printf("3:%d",sigsetjmp(jmpbuf, 1/*savemask*/));//sigsetjmp会覆盖之前设置,所以信号处理完会跳转到这里[包含本行],返回siglongjmp设置的变量
    printf("enter to the sleep.\n");
    sleep(time);//阻塞时收到信号,如果信号处理函数中没有 siglongjmp ,信号处理完返回此处阻塞
    printf("sleep is over, main over.\n");
    exit(0);
}
```

## sigsuspend函数

```
#include <signal.h>
int sigsuspend(const sigset_t *sigmask);
    sigmask 为提供给内核的屏蔽信号集
    该函数的作用是在接到一个信号之前将程序挂起,
    直到接到一个信号处理函数处理完后返回
    作用是防止信号冲突程序被挂起
```
