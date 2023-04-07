# 文件

1.  修改句柄阻塞或非阻塞

    ```c
    int fd_set_blocked(int fd, int blocked) /* {{{ */
    {
     int flags = fcntl(fd, F_GETFL);

     if (flags < 0) {
         return -1;
     }

     if (blocked) {
         flags &= ~O_NONBLOCK;
     } else {
         flags |= O_NONBLOCK;
     }
     return fcntl(fd, F_SETFL, flags);
    }
    ```
2. C标准文件操作
   1.  计算文件大小

       ```c
       #include <stdio.h>
       #include <stdlib.h>
       int fsize(const char * filename);
       int main(){ 
        //使用示例 
        printf("%d",fsize("main.cpp")); 
        return EXIT_SUCCESS;
       }
       /**
       * 计算文件大小
       * @param char * filename 文件名
       */
       int fsize(const char * filename){ 
        FILE* f=fopen(filename,"r"); 
        int filelen; 
        fseek(f,0,SEEK_END); //文件指针移到末尾 
        filelen=ftell(f); //获得文件当前指针位置，即为文件长 
        fclose(f); 
        return filelen;
       }
       ```
   2.  读取文件成字符串:

       ```c
       #include <stdio.h>
       #include <stdlib.h>
       #include <string.h>
       /**
       * 读取文件内容
       */
       char * file_get_contents(const char* file){ 
        FILE * fh= fopen(file,"rb"); 
        if(fh==NULL){ 
            printf("can’t open file no:%d",errno); 
            return NULL; 
        } 
        fseek(fh,0,SEEK_END); 
        size_t flen= ftell(fh); 
        char* filestr= (char*)malloc(flen+1);     
        memset(filestr,'\n',flen); 
        rewind(fh); 
        fread(filestr,flen,1,fh); 
        fclose(fh); 
        return filestr;
       }
       /**
       * 写内容到文件
       */
       size_t file_put_contents(const char*file,char*filestr,size_t filelen){ 
        FILE * fh=fopen(file,"w"); 
        if(fh==NULL){ 
            printf("can’t open file no:%d",errno); 
            return NULL; 
        } 
        size_t wlen=fwrite(filestr,filelen,1,fh); 
        fclose(fh); 
        return wlen;
       }
       ```
3.  C文件函数

    ```c
    //标准IO操作 <stdio.h>
    //
    sprintf(str*,const *str,...);//其他类型转为字符串,或字符串格式化,第一个参数为转后保存指针
    snprintf (char *, size_t, const char *, ...);//安全的版本,介绍见上一行
    fprintf(FILE *file,const *str,...);//格式化输出到文件
    printf(const *str,...);//格式化输出控制台
    //变量参数列表vs_list 配套的上面三函数,va_list为strarg中,见上面的strarg.
    vsprintf(str*,const *str,va_list);//其他类型转为字符串,或字符串格式化,第一个参数为转后保存指针
    vasprintf(str**, fmt, *fmt,va_list);//将va_list的字符串为一个字符串,并返回该字符串的指针,onlylinux
    vsnprintf (char *, size_t, const char *,va_list);//安全的版本,介绍见上一行
    vfprintf(FILE *file,const *str,va_list);//格式化输出到文件
    vprintf(const *str,va_list);//格式化输出控制台
    sscanf (const char*, const char*, ...);//字符串转化为其他类型,参数:字符串,格式化字符,输出值指针
    fscanf (FILE*, const char*, ...);//格式化文件读取
    scanf (const char*, ...);//控制台输入
    //
    //FILE* 结构体,为文件打开指针,标准输入输出等均为该类型.(大部分平台为整形值)
    FILE* f=fopen("filename","a");//打开文件
    FILE* f=freopen("filename","a",stdin);//重新向打开流
    //文件内操作
    fseek (FILE*, long, int);//定向文件流的指针到指定位置 参数 :文件指针,偏移位置(long),位置:SEEK_SET,SEEK_CUR,SEEK_END
    ftell (FILE*);//得到文件流指针的位置,long返回值
    rewind (FILE*)//定向文件流指针到文件头部 = fseek(FILE*, 0, SEEK_SET);
    fgetpos    (FILE*, fpos_t*);//取得文件流指针的位置
    fsetpos    (FILE*, fpos_t*);//设置文件流指针的位置
    feof (FILE*);//文件流是否到了文件底部
    //文件内操作
    fflush(FILE* f);//缓冲区的内容强制刷入文件
    fclose (FILE*)//关闭流
    ferror (FILE*);//操作文件发生错误时候使用
    clearerr (FILE*);//清理操作文件时候发生的错误
    rename(const char *,const char*);//改文件名
    remove(const char *);//删除一个文件
    FILE* f=tmpfile();//得到一个临时文件
    tmpnam(char *tempname);//得到一个唯一的文件名
    setbuf(FILE *f,char *);//设置缓存区,进行FILE IO的时候缓存文件数据保存地方
    //前两参数跟上面一样,mode 为缓存模式(_IOFBF,_IOLBF,_IONBF),size 为缓存大小
    int setvbuf(FILE *stream, char *buf, int mode, unsigned size); 
    //以下两函数二进制安全,如直接写数组等
    fread(void *,size_t 数据块大小,size_t 数据块数量,FILE* f);//读文件
    fwrite(const void*, size_t 数据块大小, size_t 数据块数量, FILE*);//写文件
    ```
