# 汇编寄存器

> dos debug命令

打印内存 d\
修改指令 a\
查看指令 u\
修改内存 e\
执行指令 t 查看寄存器 r

> gdb 命令 disas 地址 显示指定位置汇编 = u i r (info reg) 显示寄存器 = r i r a 显示所有寄存器 i r rax 显示指定寄存器 x/32x 显示指定内存 = d si 执行 ni 按步执行 = t
>
> 通用寄存器

a 累加 常用运算\
b 基址 存放地址\
c 计数 循环设置cx 后loop\
d 数据 数据传递

规则: 把a代替\
ah 高8位\
al 低8位\
ax 16位 = ahal eax 32位\
rax 64位

> 段和偏移地址寄存器

ss 栈段地址 sp 栈顶地址 push pop\
bp 函数时的栈低地址

ds,es 内存指针段 ds\[0] 为具体的内存位置,配合下面\
di,si 变址指针 做偏移时用 如ds\[si],es\[di]

ip 代码指针 执行保存代码 加代码字节 后执行\
cs 代码段

> 标记寄存器

OF 运算溢出(1)\
DF 指针寄存器方向 用在 movsb 等拷贝时增加或减少,cld清除 std设置 IF 中断(1)\
TF 调试\
SF 负值(1)\
ZF 运算结果0(1)

1.  mov ax,1 &#x20;

    sub ax,0 相减并修改ZF &#x20;
2.  mov ax,1 &#x20;

    cmp ax,ax 测试相减0修改ZF &#x20;
3.  mov ax,0 &#x20;

    mov dx,1 &#x20;

    test ax,dx 测试并运算为0修改ZF

AF 辅助进位(1)\
PF 为1位奇偶 1(偶)\
CF 运算进位(1)
