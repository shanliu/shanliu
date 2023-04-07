#vim


常用插件 

> NERDTree TagList WinManager Ctags MiniBufExplorer

打开新文件

> :open file :new file

新窗口打开文件

> :split file

切换编辑窗口

> Ctrl +ww [Ctrl +wj 下 Ctrl +wk 上]

十六进制查看文件

> :%!xxd

关闭十六进制查看文件

> :%!xxd -r

查找下一个

> /text

查找上一个

> ?test

高亮查找

> :set hlsearch

逐步查找

> :set incsearch

忽略大小写

> :set ignorecase

替换第一个匹配[正则]

> s/old/new/

替换所有行第一个匹配[正则]

> %s/old/new/

指定函数匹配

> :1,$ /old/new/

移动到文件头行

> [[

移动到文件尾行

> ]]
> [{ -- 转到上一个位于第一列的"{"
> }] -- 转到下一个位于第一列的"{"

撤销

> u

重做

> Ctrl+r

删除行

> dd

删除指定行

> :1,$d

复制指定行到指定行之后

> :1,3 co 5

移动指定行到指定行之后

> :1,3 m 5

拷贝行

> yy

粘贴

> p

剪切

> x

进入查看模式

> v

重新打开

> :e!

保持并关闭

> x

执行命令

> :!command

显示非打印字符

> :set list

TAB缩进

> < 和 >

自动缩进当前行

> ==

命令历史

> q:`选中回车执行`


设置窗口宽度

> :resize +100 :resize -100

设置窗口宽度

> :vertical resize +100

TAB操作 新标签打开

> :tabe

标签关闭

> :tabc

查看所有TAB

> :tabs

关闭其他TAB

> :tabo

进入上一个TAB

> :tabp

进入下一个TAB

> :tabn

编译

> :make

查看编译错误

> :cc 

buffer切换[上下打开文件切换] 切换文件-下一个文件

> :bn

切换文件-上一个文件

> :bp

切换到指定文件

> :b# 

删除#编号buffer

> :bdelete #

##### CTAG 命令

> Ctrl+] 前往定义

> Ctrl+t 返回

> Ctrl+o 前进

TagList 命令

```
打开
:Tlist
查看详细
回车
新窗口查看详细
o
查看原型
空格
更新taglist窗口中的tag
u
更改排序方式，在按名字排序和按出现顺序排序间切换
s
taglist窗口放大和缩小，方便查看较长的tag
x
打开一个折叠
zo
将tag折叠起来
zc
打开所有的折叠
zR
将所有tag折叠起来
zM
```

WinManager 命令

```
打开 e 后面跟目录或文件
:e ./
-    返回上级目录
c    切换vim 当前工作目录正在浏览的目录
d    创建目录
D    删除目录或文件
i    切换显示方式
R    文件或目录重命名
s    选择排序方式
x    定制浏览方式, 使用你指定的程序打开该文件
```