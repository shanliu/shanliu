# 语法

1. 引入文件
   *   系统头文件 \[常用头文件]

       ```c
       //C头
       #include <stdio.h> //IO操作
       #include <stdlib.h>  //内存分配，随机数，排序等常用函数
       #include <time.h> //时间操作
       #include <stdarg.h> //不定参数
       #include <signal.h> //信号
       #include <setjmp.h> //异常处理函数，如信号跳转点函数 sigsetjmp
       #include <sys/wait.h> //进程结束wait waitpid 等函数
       #include <unistd.h> //系统调用，pipe，fork，getpid，IO（read,write,close等）
       #include <fcntl.h> //open等底层函数
       #include <sys/types.h> //基本类型定义
       #include <sys/stat.h> //文件状态信息
       #include <string.h> //字符操作函数 如strcpy strlen等函数
       #include <sys/socket.h> //网络操作函数 如：socket inet_pton listen 等函数
       #include <pthread.h> //线程处理函数
       #include <ctype.h> //字符处理 如 tolower toupper 等
       #include <assert.h> //断言函数
       #include <errno.h> //错误码定义
       #include <limits.h> //常用数据类型最值
       #include <math.h> //常用数学操作函数
       #include <wchar.h> //宽字符处理，类似string.h
       #include <wctype.h> //宽字符处理，类似ctype.h
       #include <complex.h>　　 //复数处理
       #include <fenv.h>　　　　//浮点环境
       #include <inttypes.h>　　//整数格式转换
       #include <stdbool.h>　　 //布尔环境
       #include <stdint.h>　　　//整型环境
       #include <tgmath.h>　　　//通用类型数学宏
       //C++头
       #include <algorithm>　　　 //STL 通用算法
       #include <bitset>　　　　　//STL 位集容器
       #include <cctype>         //字符处理
       #include <cerrno> 　　　　 //定义错误码
       #include <cfloat>　　　　 //浮点数处理
       #include <ciso646>         //对应各种运算符的宏
       #include <climits> 　　　　//定义各种数据类型最值的常量
       #include <clocale> 　　　　//定义本地化函数
       #include <cmath> 　　　　　//定义数学函数
       #include <complex>　　　　 //复数类
       #include <csignal>         //信号机制支持
       #include <csetjmp>         //异常处理支持
       #include <cstdarg>         //不定参数列表支持
       #include <cstddef>         //常用常量
       #include <cstdio> 　　　　 //定义输入／输出函数
       #include <cstdlib> 　　　　//定义杂项函数及内存分配函数
       #include <cstring> 　　　　//字符串处理
       #include <ctime> 　　　　　//定义关于时间的函数
       #include <cwchar> 　　　　 //宽字符处理及输入／输出
       #include <cwctype> 　　　　//宽字符分类
       #include <deque>　　　　　 //STL 双端队列容器
       #include <exception>　　　 //异常处理类
       #include <fstream> 　　　 //文件输入／输出
       #include <functional>　　　//STL 定义运算函数（代替运算符）
       #include <limits> 　　　　 //定义各种数据类型最值常量
       #include <list>　　　　　　//STL 线性列表容器
       #include <locale>         //本地化特定信息
       #include <map>　　　　　　 //STL 映射容器
       #include <memory>         //STL通过分配器进行的内存分配
       #include <new>            //动态内存分配
       #include <numeric>         //STL常用的数字操作
       #include <iomanip> 　　　 //参数化输入／输出
       #include <ios>　　　　　　 //基本输入／输出支持
       #include <iosfwd>　　　　　//输入／输出系统使用的前置声明
       #include <iostream> 　　　//数据流输入／输出
       #include <istream>　　　　 //基本输入流
       #include <iterator>        //STL迭代器
       #include <ostream>　　　　 //基本输出流
       #include <queue>　　　　　 //STL 队列容器
       #include <set>　　　　　　 //STL 集合容器
       #include <sstream>　　　　 //基于字符串的流 stringstream等
       #include <stack>　　　　　 //STL 堆栈容器
       #include <stdexcept>　　　 //标准异常类
       #include <streambuf>　　　 //底层输入／输出支持
       #include <string>　　　　　//字符串类
       #include <typeinfo>        //运行期间类型信息
       #include <utility>　　　　 //STL 通用模板类
       #include <valarray>        //对包含值的数组的操作
       #include <vector>　　　　　//STL 动态数组容器
       ```
   *   自定义头文件

       ```c
       #include <comst.h>
       #include <comst.hpp>
       ```
2. 宏 常量 变量
   *   宏

       ```c
       // 
       #ifndef PI
       #define S(a,b) a+b;/*定义宏*/
       #endif
       #define a1(b) #b // 一个#号说明b被当成一个字符串 结果为 "b"
       #define a2(b) s##b //两个#号表示b被当成字符串且与字符s链接
       #define a3()  do{ }while(0) //复杂定义使用,保证任何场合都适用
       ```
   *   常量

       ```c
       const int v1 = 100;定义类型常量
       ```
   *   变量定义问题

       ```c
       register int i;//i会放入寄存器
       ////////////////////////////////////////////////////////////////
       //struct 的声明可以省略typedef,注意事项:
       //内存对齐规则
       //结构体大小必须是结构体中最大的类型的倍数
       //结构成员的起始位置必须是该类型(基本数据类型)的大小的倍数
       //结构体省内存方法
       //成员小的放置于前面,相同大小的尽量放置于一块,成倍数的放置于相邻位置
       ////////////////////////////////////////////////////////////////
       //结构体1
       struct struct1
       {
       int day;
       };
       //
       //结构体2
       typedef struct{
       char c;
       } struct2;
       //struct2 *p;
       //p=(struct2 *)malloc(sizeof(struct2));//动态分配内存
       ////////////////////////////////////////////////////////////////
       //联合1
       union var{  
       long int l;  
       int i;  
       };
       //
       //联合2
       typedef union{
       char c;
       int a;
       } var1;
       ////////////////////////////////////////////////////////////////
       //enum 是整形,且不会有类型验证
       enum e1{HI}; 
       enum e2{GOOD};
       enum e1 a=HI;
       enum e2 b=GOOD;
       //a==b true
       ////////////////////////////////////////////////////////////////
       //函数指针
       typedef void(*fn)(void *m);
       void hi(void *m){
       printf("%s","hi");
       };
       fn b=&hi;
       ////////////////////////////////////////////////////////////////
       //空指针 void*
       void* k=(void *)(0);
       ////////////////////////////////////////////////////////////////
       //数组长度
       int a[]={1,2};//a 为数组名，非指针，但数组名有类指针功能
       int len=sizeof(a)/sizeof(a[0]);
       ```
   *   自动变量类型

       ```c
       ////////////GUNC///////////////
       int a=1;
       typeof(a) b=a;
       printf("%d",b);
       //////////C11////////////
       int a=1;
       auto b=a;
       printf("%d",b);
       ```
   *   变量作用域和优先级

       ```c
       //文件内变量，文件外不可见
       static int w1=100;
       //C++中，局部变量存在与{}中
       bool a=true;
       while(a){
       int i=1;
       a=false;
       }
       cout << i;//未定义
       for(int i=1;i<2;i++){}
       cout<< i;//未定义
       ////////////////////////////////////////////////////////////////
       //引用其他文件定义，即其他文件有定义 int a;
       extern int a;
       ////////////////////////////////////////////////////////////////
       //++问题
       int t1 = 5, t2 = 7;
       t1+++t2;//12 t1++ +t2 ++后为执行后加 ++前为执行前加
       ```
   *   变量引用 1. int& 表示一个引用,引用看着操作对象的别名 2. 引用跟引用对象的内存地址相同

       ```c
       int k=10;
       int&rk=k;
       //&k==&rk
       ```

       1.  引用定义时初始化，创建时必须绑定操作对象

           ```c
           int k=10;
           //int&b;
           //b=k;wrong
           int& b=k;
           ```
       2.  形参引用为传引用

           ```c
           int a(int &b){
           return 1;
           }
           int main(){
           int b=2;
           a(b);
           }
           ```
       3.  返回为以引用时候，可以把返回值赋值

           ```c
           struct a{
           int id;
           }
           a va;
           a & fn(){
           return va;
           }
           int main(){
           fn().id=10;
           return 0;
           }
           ```

*   字符数字转换

    ```c
    /////////////////////////数字转字符//////////////////////////////
    //-----------------------------C-------------------------------
    //C方式1 标准方式
    #include <stdio.h>
    int n=1000;
    char ns[5];
    snprintf(ns,5,"%d",n);
    printf("%s",ns);
    //
    //C方式2,浮点数
    double value= 86.789e5;
    int dec, sign;
    int ndig = 5;
    //参数:浮点数,要转换的位数,小数点的位置(返回),正或负数(返回)
    //返回:连续的字符串,无小数点
    char *p = ecvt(value,ndig,&dec,&sign);
    //跟上面一样,但结果会四舍五入
    char *p = fcvt(value,ndig,&dec,&sign);
    //
    //C方式3
    string[10];
    //整数,存储的字符串,整数进制(2,10等) 后面没有\0结束
    itoa(int, string*, 10);
    //
    //-------------------------C++-------------------------------
    //C++方式1
    #include <string>
    string sinta("1234");
    const char *tchar=sinta.c_str();
    //
    //C++方式2
    #include <string>
    #include <sstream>
    int inta=12345;
    stringstream tsinta;
    tsinta << inta;
    string sinta =tsinta.str();
    //或者，注意流的 << >> 重载
    string sinta; 
    tsinta >> sinta;
    //
    /////////////////////////字符转数字//////////////////////////////
    //C方式 1
    #include <stdlib.h>
    float fa=atof("10.1");//字符串转浮点
    int ia=atoi("10");//字符串转整形
    long int la=atol("10");//字符串转整形
    //
    //C方式 2 标准方式
    #include <stdio.h>
    int a;
    sscanf("2006", "%d", &a);//字符转数值
    //C++方式
    #include <string>
    #include <sstream>
    int a;
    string str = "-1024";
    istringstream issInt(str);
    issInt >> a;
    ```

1. 函数 类
   * 函数 \[容许重名]
     1. 先根据函数名查找
     2. 根据类型加数量匹配
     3. 模糊匹配，类型转换

```c
  //头文件
  typedef void(*fnp)();
  void hi();
  fnp fn();
  //实现
  void hi(){
   printf("%s","hi");
  };
  fnp fn(fnp){
   return hi;
  }
  //使用
  fnp c=fn(hi);
  c();
  ///////////////////////////模板/////////////////////////
  //头文件
  template<typename T> void swap(T& t1, T& t2);
  //实现
  template<typename  T> void swap(T& t1, T& t2) {
     T tmpT;
     tmpT = t1;
     t1 = t2;
     t2 = tmpT;
  }
  //使用
  int t1=1;
  int t2=2;
  swap<int>(t1,t2);
```

*   类 1. 基本定义

    ```c
    //头文件 hpp
    struct test{
    public:
       test();
       int number1;
       static int number2;
       void fn1();
       static void fn2();
       ~test();
       inline void inlinefn(){
         //inline 必须写到头文件中
       };
    protected:
       char* number3;
       static char number4[10];
       void fn3();
       static void fn4();
    private:
       int number5;
       static char* number6;
       void fn5();
       static void fn6();
    };
    //实现
    int test::number2 = 1;
    char test::number4[10]="init char";
    char* test::number6=NULL;
    test::test():number1(10),number5(10){//初始化成员值
    //构造函数
    };
    void test::~test(){
    //析构函数
    }
    void test::fn1(){
     printf("%d",this->number5);
    }
    void test::fn2(){
    }
    void test::fn3(){
    }
    void test::fn4(){
    }
    void test::fn5(){
    }
    void test::fn6(){
    }
    ```

    1.  修饰符

        ```c
        void mb(){
        }
        class objvar{
        public:
        objvar(int a,int b){
        }
        }
        class obj{
        public:
        obj (const obj& objvar){
        //拷贝构造函数,一般有内存拷贝时实现
        //声明时：obj t=varobj 或 obj t(varobj) 调用
        //动态创建：obj* t=new obj(varobj);
        //参数传递：void test(obj t){} 参数调用时
        }
        obj():a(10),b(10),ob(1,2){//构造函数 初始化 成员变量
        //一定会执行
        }
        obj(int a,int b=10){
        }
        objvar ob;
        int a;
        void ma(){
        mb();//=this->mb();
        ::mb();//使用全局函数
        return a+b;//= this->a+this->b;
        }
        virtual ～obj(){//加上virtual 有利于释放资源
        //free res
        }
        protected:
        int b;
        virtual void mb(){}
        private:
        int c;
        }
        strcut tobj{//抽象类
        public:
        virtual void vmb(int& b)=0;//纯虚函数
        }
        struct cobj : public obj,public tobj{
        //public private protected 继承方式
        //支持多重继承
        public:
        cobj(){
        obj::mb();//call parent method
        }
        ～cobj(){
        //free res
        }
        void vmb(int& b){
        //实现纯虚函数
        }
        protected:
        void mb(){
        //当子类对象转为父类指针调用时候
        //virtual 函数会调用此方法，否则调用父方法
        }
        }
        int main(){
        int b(10); // int b=10;
        int t(b);//= int t=b;
        obj a(10,20);
        obj*pa=new a(10,30);
        delete pa;
        int*pb= new int[1024];//连续的对象申请
        delete [] pb;
        return 0;
        }
        ```
    2.  操作符重载

        ```c
        class a{
        public:
        int t[10];
        a operator +(const a& avar);//+-* /
        //包含多个元素，方便操作,返回引用方便操作左值
        int& operator [](int index);
        //比较对象，avar可以是非class a
        bool operator ==(const a& avar);
        //类型转换
        operator int();
        operator const char*();
        //输入输出
        a& operator <<(int b);//输出
        //privae ~~
        private:
        string a("aaa");
        }
        //对class a的+操作进行重载
        //实现 operator+
        a::operator+(const a& var){
        //code~~
        }
        //实现 operator[]
        int& a::operator[](int index){
        return t[index];
        }
        //实现 operator==
        bool a::operator ==(const a& avar){
        return true;
        }
        //实现 operator type
        a::operator int(){
        return 1;
        }
        a::operator const char*(){
        return a.c_char();
        }
        //实现 operator <<
        a& a::operator<<(int a){//输出
        printf("%d",a);
        return *this;
        }
        //全局操作符重载
        //全局操作符重载 对全局的+操作进行重载
        a operator+(const a& vara,const a& varb ){
        //code ~~
        }
        //两种都存在时候，会调用类操作符
        int main(){
        a vara;
        a varb;
        vara+varb;//operator +
        vara[1];//operator []
        vara==varb;//operator ==
        int t =(int)vara;//operator ()
        varb <<1;//operator <<
        return 0;
        }
        ```
    3.  内部类

        ```c
        class a{
        public:
        class b{
        public:
        void t();
        }
        }
        //
        void a::b::t(){
        printf("out");
        }
        int main(){
        new a::b();
        }
        ```
    4.  模板类

        ```c
        //头文件
        template <class T>class ttest{
        public:
        ttest();
        ~ttest();
        void sett(T t);
        T gett();
        private:
        T *m_pT;
        };
        //实现
        template <class  T>  ttest<T>::ttest(){
        m_pT = new T[1];
        }
        template <class T>  ttest<T>::~ttest() {
        delete [] m_pT ;
        }
        template <class T> void ttest<T>::sett(T t) {
        m_pT[0] = t;
        }
        template <class T> T ttest<T>::gett() {
        T t = m_pT[0];
        return t;
        }
        //使用
        ttest<int> tvar;
        tvar.sett(1);
        printf("%d",tvar.gett());
        ```
    5.  类函数指针

        ```c
        //头文件
        class ru_m {
        public:
        ru_m();
        typedef int (ru_m::*p)();
        p get_m();
        int show();
        };
        //实现
        int ru_m::show(){
        return 10000;
        }
        ru_m::p ru_m::get_m(){
        ru_m::p vc;
        //不能是this->show c++ 中对象成员函数为偏移非指针
        vc=&ru_m::show;
        return vc;
        }
        //使用
        int main()
        {
        ru_m r;
        ru_m::p p=r.get_m();
        printf("%d",(r.*p)());
        return 0;
        }
        ```
    6.  模板关键字typename和class

        > 1. 模板定义语法中关键字class与typename的作用完全一样。
        > 2.  除class和typename外还可以用其他类型定义模板,如以下
        >
        >     ```c
        >     //头文件
        >     template<typename T,int MAX> void testtemplate(T t1);
        >     //实现
        >     template<typename  T,int MAX> void testtemplate(T t1) {
        >     T b[MAX];
        >     for(T i=MAX;i>0;i--){
        >     b[i]=t1;
        >     printf("%d",b[i]);
        >     }
        >     }
        >     //使用
        >     int t1=1;
        >     testtemplate<int,10>(t1);
        >     ```

1.  名字空间

    ```cpp
    namespace myname{
    class a{}
    }
    namespace myname1{
    class a{}
    }
    using myname1::a;//单个
    using namespace std;//全部
    int main(){
    myname::a avar myname::a;
    a bvar a;
    cout <<"hi";
    }
    ```
