# 动态加载库

1. 创建库 b.cpp -fPIC 生成于位置无关的代码 实现不同机器共享

```c
extern "C"{//加上,防止进行CPP编译器编译 导致找不到
    int add(int a,int b)
    {
        return (a + b);
    }
}
```

> gcc -shared b.cpp -o dny.so -fPIC

1. 使用库 a.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>
typedef int (*CAC_FUNC)(int, int);
int main()
{
    void *handle;
    char *error;
    CAC_FUNC cac_func = NULL;
    handle = dlopen("./dny.so", RTLD_LAZY);
    if (!handle) {
        fprintf(stderr, "%s\n", dlerror());
        exit(EXIT_FAILURE);
    }
    *(void **) (&cac_func) = dlsym(handle, "add");
    if ((error = dlerror()) != NULL)  {
        fprintf(stderr, "%s\n", error);
        exit(EXIT_FAILURE);
    }
    printf("add: %d\n", (*cac_func)(2,7));
    dlclose(handle);
    exit(EXIT_SUCCESS);
}
```

> gcc -o main a.c -ldl
