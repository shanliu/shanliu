# 管道

```
//管道,管道适用IO操作,用于父子,或邻居进程间的通信,单通道.
int pipe_fd[2]; //存储管道指针 :0表示读管道,1表示写管道
if(pipe(pipe_fd)<0)//返回值小于0说明管道创建失败
//单独管道创建没有意义,必须配合fock等进程创建函数
pid_t pid;
if((pid=fork())==0){//派生的时,会产生内存拷贝,而管道,网络连接等会拷贝一个指针副本,系统管理计数.
 //此处表示子进程
 close(pipe_fd[1]);//关闭写管道
 sleep(3);//确保父进程管道内信息
 r_num=read(pipe_fd[0],r_buf,100);
 printf( "read num is %d the data read from the pipe is %d ",r_num,atoi(r_buf));
 close(pipe_fd[0]);//关闭读管道
}else if(pid>0){
 //此处表示父进程,父进程可以得到子进程ID
 close(pipe_fd[0]);//关闭父端的读管道
 strcpy(w_buf,"111");
 if(write(pipe_fd[1],w_buf,4)!=-1)//写管道信息,等待子进程去读取 printf("parent write over "); close(pipe_fd[1]);//关闭写管道
 sleep(10);//等待子进程读完数据并写出来
}
pread (int __fd, void *__buf, size_t __nbytes,__off_t __offset);//
pwrite (int __fd, __const void *__buf, size_t __n,__off_t __offset);
pipe (int __pipedes[2]);//通道
```
