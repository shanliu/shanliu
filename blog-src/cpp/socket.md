# 网络

```c
#include <netdb.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
int main(int arg,char *args[])
{

        struct addrinfo k,*pk;

        memset(&k,0,sizeof(struct addrinfo));
        k.ai_family=AF_UNSPEC;
        k.ai_socktype=SOCK_STREAM;
        k.ai_flags=AI_PASSIVE;
        char o[100];
        int gai;
        char *r;
        if(arg<=1){
                exit(0);
        }
        /*第4个参数是指向addrinfo结构指针的指针
        原理:getaddrinfo函数会根据传入的条件申请内存并创建addrinfo结构体
        在根据第4个参数传入的指针的指针,得到指向指向addrinfo的指针,并修改该值指向创建好的结构体
        因为在函数内部申请的内存,所以不需要addrinfo的时候需要调用freeaddrinfo()释放内存;*/;
        gai=getaddrinfo(args[1],"80",&k,&pk);
        if(gai!=0){
                printf("error:%s",gai_strerror(gai));
                exit(0);
        }
        printf("domain:%s\n",args[1]);
        void * sockaddr=NULL;
        while(pk){
                switch(pk->ai_family){
                        case AF_INET:
                                sockaddr=&((struct sockaddr_in *)pk->ai_addr)->sin_addr;//sockaddr_in.sin_addr 是结构体 ,不是结构体指针
                                break;
                        case AF_INET6:
                                sockaddr=&((struct sockaddr_in6*)pk->ai_addr)->sin6_addr;//sockaddr_in.sin6_addr 是结构体 ,不是结构体指针
                                break;
                }
                if(sockaddr){
                        inet_ntop(pk->ai_family,sockaddr,o,100);//第2个参数需要的是sockaddr指针,不是结构体
                        if(r!=NULL){
                                printf("ip address:%s & ip type:%d\n",o,(pk->ai_family==AF_INET)?4:6);
                        }

                }
                sockaddr=NULL;
                pk=pk->ai_next;
        }

        freeaddrinfo(pk);
}
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>

#define LEN 100
void str_to(int cs,struct sockaddr * addr,int addr_len){
        char a[LEN]="aaaaa\n";
        //read(cs,a,LEN);
        printf("%s",a);
        int i=1000;
        for(;i>0;i--){
                write(cs,a,strlen(a));
                sleep(1);
        }
}

void sigpro(int *sig){
        pid_t t;
        while((t=waitpid(-1,NULL,WNOHANG))>0){
                printf("c:%d\n",t);
        }
}

int main()
{

        signal(SIGCHLD,&sigpro);
        int s,cs;
        s=socket(AF_INET,SOCK_STREAM,0);
        struct sockaddr_in add,cadd;
        memset(&add,0,sizeof(add));
        memset(&add,0,sizeof(cadd));
        add.sin_family=AF_INET;
        add.sin_port=htons(8012);
        inet_pton(AF_INET ,"127.0.0.1", &add.sin_addr);//sockaddr_in.sin_addr 是结构体 ,不是结构体指针
        bind(s,(struct sockaddr*)&add,sizeof(add));
        listen(s,5);
        while(1){
                int clen;
                cs=accept(s,(struct sockaddr*)&cadd,&clen);
                pid_t t;
                t=fork();
                if(t==0){
                        str_to(cs,(struct sockaddr *)&cadd,clen);
　　　　　　　　　　　　　　close(cs);
                        exit(1);
                }else{
　　　　　　　　　　close(cs);
　　　　　　　　}
        }
        return 1;
}
```

```c
struct pollfd fds[CMAX];

        int in= fileno(stdin);
        int out= fileno(stdout);

        fds[0].fd=in;
        fds[0].events=POLLIN;// 可以多个事件 POLLIN|POLLOUT

        char a[100];
        memset(a,0,100);

        fds[1].fd=out;
        fds[1].events=POLLOUT;

        FILE *f;
        f=fopen("./txt","a");

　　　　enum {
　　　　　　SN,
　　　　　　SB
　　　　} S;
　　　　S=SN;
        while(1){
　　　　　　if(S==SB)
　　　　　　　　break;
                if( poll(fds,CMAX,0)>0){
                        int i=0;
                        for(;i<CMAX;i++){
                                if(fds[i].revents==POLLIN){//判断返回事件
                                        read(in,a,100);

                                        if(a[0]=='q'){
                                                printf("%s","fasdfas");

　　　　　　　　　　　　　　　　　　　　　　　　　　　　S=SB;

                                        }

                                        printf("in:%s",a);
                                        fwrite(a, strlen(a), 1, f);
                                }
                        }
                }

        }    
 fclose(f);
        return EXIT_SUCCESS;
```
