# 调试与错误

1.  有定义,无实现错误:(C/C++混编出现此错,在头文件(\*.h)添加extern "C"{})

    > undefined reference to '_\*_';
    >
    > ```c
    > int a();
    > a();
    > ```
2.  函数未实现,被调用:

    > is used uninitialized in this function \*\*
    >
    > ```c
    > typedef void (*b)();
    > b a;
    > a();
    > ```
3.  函数未在main之前声明.

    > error: conflicting types for \*
4.  参数传递问题

    > 函数参数传递指针时,函数为为指针的拷贝 所以要修改指针时,需要传递指针的指针
5.  编译问题

    > 注意 GCC 会根据编译文件的文件名决定使用C++还是C

    *   GCC 编译生成汇编

        > gcc -S cfile
    *   GCC 编译不连接文件

        > gcc -c cfile
    *   GCC 编译生成优化汇编

        > gcc -O2 -S cfile
    *   .o 文件反编译

        > objdump -d ofile.o
    *   GDB 调试选项

        > gcc -g cfile
    *   GCC 生成库

        > gcc -shared b.cpp
6.  CDT 控制台问题

    > \-mwindows 编译参数加表示无控制台 -mconsole 表示控制台程序 默认不加,两个都有...
7.  动态内存问题

    > 所有一般需要大内存的时候会使用malloc或者calloc来得到堆区内存 有时候在函数内返回一个指针什么的,使用malloc,别返回栈区的指针
    >
    > 除了编译期可以知道字符串大小的,字符串指针可以不用声明长度 其他情况下字符串需声明长度
    >
    > 一般结构体,联合,整形等都是编译期就知道大小的,所以函数返回的时候返回其拷贝 但char\[]虽然知道长度,但返回指针,因为其是在运行期才分配. char_,int_等指针返回为指针的拷贝.
8.  GDB调试

    GDB标准库查看脚本，下载并为:\~\\/.gdbinit

    > [http:\\/\\/www.yolinux.com\\/TUTORIALS\\/src\\/dbinit\_stl\_views-1.03.txt](http://www.yolinux.com/TUTORIALS/src/dbinit\_stl\_views-1.03.txt)

    运行

    ```
    gdb ./a.out
    //或
    gdb php -c core xx.core
    ```

    GDB命令及注释

    > gdb -tui app

    ```
    r 运行
    s 单步执行
    n 下一行执行
    si 汇编步行
    ni 汇编下一行
    return 强制函数返回,后面代码不运行
    finish 完成函数
    b 设置断点(可以函数名 行数)
    info b 查看断点
    bt 显示堆栈
    set args 设置参数
    c 继续运行
    p 打印变量
    pt 打印类型
    x/32t x/32t 16/2进制显示32个连续内存,类似上面p
    pvector 等打印STL
    watch 当变量修改断点
    list  查看当前运行行的上下文代码
    layout src 显示源码
    layout asm 显示汇编
    layout regs 寄存器和汇编
    layout split 汇编和源码
    fs next 切换tui
    disas 查看汇编
    info all-registers 查看所有寄存器
    set follow-fork-mode child 设置fork调试进入子进程
    t 线程和进入指定线程列表
    target remote ip:1234 连接到指定GDB SERVER调试 调试前使用 file *.o 文件加载源码
    ```

    ```
    //循环打印
    set $i=10;
    while($i--)
    p $i //替换你要打印的内容
    end
    ```

> 在gdb 中查看为 x/32x 0x7fffffffe3e0-0x14 其中 0x7fffffffe3e0 为rbp -0x14 为对应栈变量偏移 例如汇编:movl $0x1,-0x14(%rbp) 表示把1设置为变量(-0x14)

1.  gdb 错误

    a. Missing separate debuginfos, use: debuginfo-install 1. 需要先修改“\\/etc\\/yum.repos.d\\/CentOS-Debuginfo.repo”文件的enable=1； 2. 使用 sudo yum install glibc 安装； 3. 使用 debuginfo-install glibc-2.12-1.132.el6.i686 安装。
2. top查看进程运行情况 1. 先用ps 得到进程ID 或用 pidof 得到 2. 在用 top -p pid 查看
3.  Makefile 示例

    ```
    a : a.o b.o
          cc -o a a.o b.o
    b.o : b.c b.h
          cc -c -g b.c b.h
    a.o : a.c
          cc -c -g a.c b.h
    clean :
          rm a a.o b.o
    #makefile dome
    ```
4. core dump 文件生成\[以下针对当前登录]
   1.  设置core dump 文件大小

       ```
       ulimit -c unlimited
       ```
   2.  设置存放目录 不设置以下为 默认为执行目录

       ```
       echo "/var/log/core-%e-%p-%t">/proc/sys/kernel/core_pattern
       //非root用户
       sudo sh -c 'echo "/var/log/core/%e-%p-%t" > /proc/sys/kernel/core_pattern'
       ```
   3.  查看 core dump 文件

       ```
       gdb exec_file_path /var/log/core_file
       ```
   4. 如果没生成core文件 检查文件夹目录权限 core生成错误提示为:Segmentation fault (core dumped)
