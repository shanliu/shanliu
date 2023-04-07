# 不定参数

1.  已知参数数量

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <stdarg.h>
    void argdome(int argnum,...);
    int main(){ 
     //使用示例 
     argdome(2,2,1); 
     return EXIT_SUCCESS;
    }
    /**
    * @param argnum 为参数数量
    */
    void argdome(int argnum,...){ 
     va_list arglist; 
     va_start(arglist,argnum); 
     for(int i=0;i<argnum;i++){ 
         //获取并转换参数类型 
         int tint=va_arg(arglist,int); 
         printf("%d\n",tint); 
     } 
     va_end(arglist);
    }
    ```
2.  未知参数数量

    ```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <stdarg.h>
    /** 
    * 未知参数数量遍历
    */
    void arg_p(char * start,...){ 
       va_list argp; 
       //start 为最后一个确定参数地址 
       va_start(argp,start); 
       char * para; 
       while(1){ 
           //从start后地址开始遍历剩余参数 
           para = va_arg( argp, char*); 
           if ( strcmp( para, "") == 0 ) 
               break; 
           printf("%s\n",para); 
       } 
       va_end(argp);
    }
    /**
    * 不定参数调用系统打印
    */
    void arg_sprint(char * fml,...){
         va_list valist;
         va_start(valist,fml);
         vprintf(fml,valist);
         //char[100] str;//存储打印字符
         //vsprintf(str,fml,valist);
         va_end(valist);
    }
    ```
