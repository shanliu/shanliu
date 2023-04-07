# 时间

1.  基本转换

    ```c
     struct tm settm; 
     strptime("2017-07-24 11:34:01", "%Y-%m-%d %H:%M:%S", &settm); 
     time_t tend =mktime (&settm); 
     time_t tstart=time( NULL ); 
     //gmtime(&tstart);//非本地时间
     printf("%d",difftime(tend,tstart));//时间差值 
     while(true){ 
         tstart = time( NULL ); 
         struct tm * tm1 =localtime(&tstart); 
         char stime[30]; 
         strftime (stime, 30, "%Y-%m-%d %H:%M:%S", tm1);
         printf("%s\n",stime); 
         fflush(stdout); 
         sleep(1); 
     }
    ```
2. 常用函数
