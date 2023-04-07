# C调用汇编

1. C代码嵌入汇编

> **asm** 是gcc宏 **volatile** 表示编译器不优化指令 如下格式:

格式:

```
__asm__ __volatile__ (
    "汇编指令" 
    : "=a" (lo), "=d" (hi) //输出
    :"d"(a),"m"(b) //输入
    :破坏
);
```

示例(使用AT\&T汇编):

```
//c 中&a 的内存地址 跟汇编里的 %ebp 寄存器加偏移一致
//c 中函数名,函数指针(c中函数在代码段地址) 跟汇编中代码地址同理 即: %eip 可以放代码地址 之后会执行该函数
unsigned int a=1,b=2,c,d;
__asm__  __volatile__( //输入输出 %从0开始 如下 c为0 d为1 a为2 d为3
"movl $11,%%eax\n"
"add %2,%%eax\n"
"add %3,%%eax\n"
"movl %%eax,%0\n"
:"=m"(c),"=a"(d)       //输出
:"d"(a),"m"(b) //输入 为什么不所有都m呢?因为指令无法内存到内存,直接用d先赋值到寄存器可以简化操作
:"%eax","%edx","memory" //告诉编译器使用了那些寄存器或内存,好像可以不传.
);
//输入辅助符 如d 会扩展并添加到这段汇编的头部:
// mov -0x8014(%rbp) %eax
// mov %eax,%edx
//输出辅助符 如a 则把%eax赋值到d中
printf("%d",c);
```

辅助符: 通用寄存器

```
"a" 将输入变量放入eax
"b" 将输入变量放入ebx
"c" 将输入变量放入ecx
"d" 将输入变量放入edx
"s" 将输入变量放入esi
"d" 将输入变量放入edi
"q" 将输入变量放入eax，ebx，ecx，edx中的一个
"r" 将输入变量放入通用寄存器，也就是eax，ebx，ecx，                                         edx，esi，edi中的一个
"A" 把eax和edx合成一个64 位的寄存器(use long longs)
```

内存

```
"m" 内存变量
"o" 操作数为内存变量，但是其寻址方式是偏移量类型，也即是基址寻址，或者是基址加变址寻址
"V" 操作数为内存变量，但寻址方式不是偏移量类型
" " 操作数为内存变量，但寻址方式为自动增量
"p" 操作数是一个合法的内存地址（指针）
```

寄存器或内存

```
"g" 将输入变量放入eax，ebx，ecx，edx中的一个或者作为内存变量
"X" 操作数可以是任何类型
```

立即数

```
"I" 0-31之间的立即数（用于32位移位指令）
"J" 0-63之间的立即数（用于64位移位指令）
"N" 0-255之间的立即数（用于out指令）
"i" 立即数  
"n" 立即数，有些系统不支持除字以外的立即数，这些系统应该使用"n"而不是"i"
```

匹配

```
" 0 " 表示用它限制的操作数与某个指定的操作数匹配，
"1" ... 也即该操作数就是指定的那个操作数，例如"0"
"9" 去描述"％1"操作数，那么"%1"引用的其实就是"%0"操作数，注意作为限定符字母的0－9 与指令中的"％0"－"％9"的区别，前者描述操作数，                                       后者代表操作数。
& 该输出操作数不能使用过和输入操作数相同的寄存器
```

操作数类型

```
"=" 操作数在指令中是只写的（输出操作数）  
"+" 操作数在指令中是读写类型的（输入输出操作数）
```

浮点数

```
"f" 浮点寄存器
"t" 第一个浮点寄存器
"u" 第二个浮点寄存器
"G" 标准的80387浮点常数
 % 该操作数可以和下一个操作数交换位置                                       例如addl的两个操作数可以交换顺序                                      （当然两个操作数都不能是立即数）                       
 # 部分注释，从该字符到其后的逗号之间所有字母被忽略
 * 表示如果选用寄存器，则其后的字母被忽略
```

1. C代码调用独立汇编文件

> c文件

```c
//gcc n.c n.S -o n
#include <stdio.h>
extern int myasmfun() asm("myasmfun");
int main(){
    int a=myasmfun(11,22,34,44,55,66,77);
    printf("%d\n",a);
    return 0;
}
```

> AT\&T汇编

```
.globl myasmfun
#if !defined( __APPLE__ ) && !defined( __FreeBSD__ )
.type  myasmfun, @function
#endif
myasmfun:

#if defined(__i386__)
    //参数: 都在栈中 pop 出来
    //返回 %eax
    push %ebp
    mov %esp,%ebp

    pop %edx
    add %edx,%eax
    pop %edx
    add %edx,%eax

    mov %ebp,%esp
    pop %ebp
    ret

#elif defined(__x86_64__)
    //参数:%edi %esi %edx %ecx %r8d %r9d 之后参数后面在栈中,pop出来
    //返回 %eax
    push %rbp
    mov %rsp,%rbp
    //sub $0x4,%rsp //函数内栈内存申请
    movl $11,%eax
    add %edi,%eax
    add %esi,%eax
    add %edx,%eax

    mov %rbp,%rsp
    pop %rbp
    ret
#endif
```
