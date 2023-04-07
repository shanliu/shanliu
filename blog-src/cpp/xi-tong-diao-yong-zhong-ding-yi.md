# 系统函数重定义

```c
#define _GNU_SOURCE //RTLD_NEXT 为GUN定义 必须.编译加-ldl
#include <stdio.h>
#include <dlfcn.h>
#include <unistd.h>
#include <sys/socket.h>

typedef int (*socket_pfn_t)(int domain, int type, int protocol);
static socket_pfn_t g_sys_socket_func;
int main(){
    //当前文件的socket定义为第一定义
    //默认调用的定义,这里为下面的socket函数
    socket_pfn_t g_sys_socket_func1 = (socket_pfn_t)dlsym(RTLD_DEFAULT,"socket");
    //这里可以获取下一个定义,即系统定义,实现修改提供函数调动
    g_sys_socket_func = (socket_pfn_t)dlsym(RTLD_NEXT,"socket");
    int sock=socket(AF_INET,SOCK_STREAM,0);
    close(sock);
}
int socket(int domain, int type, int protocol)
{
    printf("%s","hook");
    return g_sys_socket_func(domain,type,protocol);
}
```
