# ext

> export USE\_ZEND\_ALLOC=0 #禁用Zend内存管理
>
> export ZEND\_DONT\_UNLOAD\_MODULES=1 #正确显示extension的内存堆栈
>
> valgrind --leak-check=full --show-reachable=yes php test.php #valgrind 检测内存泄漏

1.  变量操作(常量)

    a. 设置变量 ZVAL\_\*系列函数;例:

    ```
     zval t;
     ZVAL_STRING(t,"10",2);
    ```

    b. 获取变量 Z_\* 系列函数_\
    _c. 获取变量指针 ZP 系列函数_\
    _d. 获取变量指针的指针 Z_\_PP 系列函数，例:

    ```
     Z_STRVAL(t);
     Z_STRLEN(t);
    ```

    e. 变量类型转换 convert_to_\* 系列函数,例:

    ```
     convert_to_long_ex(t);
    ```

    f. 获取变量类型 Z_TYPE Z\_TYPE\_P Z\_TYPE\_PP 可以同 IS_\* 系列常量对比,例:

    ```
     if(Z_TYPE_P(filehandle)!=IS_LONG)
         php_printf("this is long type!");
    ```

    g. 申请变量的内存

    ```
     //一般,宏: 
     ALLOC_ZVAL(zv);
     MAKE_STD_ZVAL(zv)
     //用完释放 
     FREE_ZVAL(zv);//等宏
     //释放变量内存 
     zval_dtor(zval);
     zval_ptr_dtor(&zval); 
     FREE_ZVAL(zval);
     //注意:FREE_ZVAL 只释放zval结构体内存,而zval_dtor会释放zval变量及value指针内存
    ```

    h. 打印普通变量

    ```
      php_printf("hi function!");
      //返回输出流,可以使用 PHPWRITE(str, str_len) 继续写
      //打印zval变量 
      php_var_dump(zval **struc, int level TSRMLS_DC);
      //var_dump C实现 在:ext/standard/php_var.h
      zend_print_zval_r(zval *expr, int indent TSRMLS_DC) 
      //print_r C实现 在Zend核心里
    ```

    i. 字符串变量生成函数

    ```
       spprintf(char **pbuf, size_t max_len, const char *format, ...); 用完释放efree();
    ```

    j. 申请内存并复制字符串的副本

    ```
       estrdup(&a)；
       estrndup(&a,10);
    ```

    k. 变量内存申请:

    ```
      emalloc(size);
      efree(ptr);
      //不要用malloc 除非你能明确释放掉
    ```

    l. 定义常量:

    ```
      REGISTER_*_CONSTANT(name, str, flags)//系列函数 
      flags CONST_* 系列宏
    ```

    m. 复制变量

    ```
      MAKE_COPY_ZVAL(ppzv, pzv)
      //申请内存并复制
      COPY_PZVAL_TO_ZVAL
      //复制一个zval指向同类型的zval
    ```
2.  全局C变量辅助宏:

    ```
     EG(v);//全局变量 executor_globals 如$_GLOBALS[EG(symbol_table),地址:EG(active_symbol_table)
     PG(v);//核心变量 php_core_globals 如:$_GET $_POST .. PG(http_globals)[TRACK_VARS_*],INI信息
     SG(v);//SAPI变量 请求数据 sapi_globals_struct 如:HTTP原始请求变量 sapi_request_info
     CG(v);//编译变量 compiler_globals 可以得到函数表,类表
     EX(v);//当前执行数据 zend_execute_data 可以获取到当前执行的函数,类,OPCODE等
     OG(v);//输出变量 output_globals
    ```
3.  函数,分5类 ZEND\_\*\_FUNCTION 以下为C定义内置函数

    函数参数说明 FOR C:

    ```
     int ht, zval *return_value, zval **return_value_ptr, zval *this_ptr, int return_value_used 
     ht 参数个数,但不能直接使用,该用 ZEND_NUM_ARGS()获取
     return_value 函数返回zval
     return_value_ptr 返回值的指针,但想控制返回值的内容,不是直接返回返回值时候用到,如参数引用实现.
     this_ptr 如果当前方法是一个对象的方法,此值指向该方法指向的对象,不能直接使用,使用 getThis() 函数
     return_value_used 返回值有没有被使用
     例:
     int argc = ZEND_NUM_ARGS();
     char *str=NULL;
     int strlen;
     //次方法用于获取函数参数
     //注意: O 参数须 &object zval 指针 和指明 zend_class_entry * 的指针
     //      s 参数需要 &str char * 指针 和 &strlen int 长度指针
     if (zend_parse_parameters(argc TSRMLS_CC, "s|s",&str,&strlen) ==FAILURE) {
       return;
     }
     //此方法一般用于类的静态方法,当函数映射类方法的时候.
     //会校验getThis()的类和object是否一个类zend_class_entry
     if (zend_parse_method_parameters(ZEND_NUM_ARGS() TSRMLS_CC, getThis(), "Os", &object, date_ce_date,&str,&strlen) == FAILURE) {
         RETURN_FALSE;
     }
    ```

    关于返回

    ```
     修改 return_value_ptr 指针指向要用ALLOC_ZVAL_*宏申请内存,
     参考:http://www.walu.cc/phpbook/6.1.md
     返回的宏 RETURN_*系列函数,资源用 ZEND_REGISTER_RESOURCE 
     RETURN_ZVAL(); 返回zval结构
         如果是申请的内存,RETURN_ZVAL(zv, copy, dtor) copy 0 dtor 0
         如果是堆栈内存 RETURN_ZVAL(zv, copy, dtor) copy 1 dtor 0
    ```
4. 数组与HASH表 1. zvalue_value 是个union 数组时为 hashtable 2. hashtable 表有很对对应的处理函数:zend\_hash系列函数 3. 数组上的HASHTABLE一般不直接操作,通过add\__系列函数处理 4. hashtable 内存申请 ALLOC\_HASHTABLE 等宏处理,用完释放FREE\_HASHTABLE 5. 数组初始化也不要用直接处理hashtable,使用 array\_init 处理
5.  资源变量 1. 需要使用资源变量须先得到一个资源列表

    参数: 1. 请求资源,每次请求会清空此请求的资源列表 2. 永久资源,服务启动后永久保存 3. 显示资源名 4. 内部管理参数,忽略

    ```
     le_myfile = zend_register_list_destructors_ex(myfile_dtor,NULL,"standard-c-file", module_number);
     static void myfile_dtor(zend_rsrc_list_entry *rsrc TSRMLS_DC){
          FILE *fp = (FILE *) rsrc->ptr;//转换指定类型,方便以后操作
          fclose(fp);
     }
    ```

    1.  把资源注册进资源列表,并设置return\_value

        ```
        int r=ZEND_REGISTER_RESOURCE(return_value, fp, le_myfile);
        ```
    2.  查询资源 filehandle 为资源zval

        ```
        ZEND_FETCH_RESOURCE(fp, FILE *, &filehandle, -1, "standard-c-file",le_myfile);
        if (fp == NULL){
        RETURN_FALSE;
        }
        ```
    3.  删除资源 filehandle 为资源zval

        ```
        if (zend_list_delete(Z_RESVAL_P(filehandle)) == FAILURE) {
         RETURN_FALSE;
        }
        ```
6.  全局函数

    ```
    zend_function_entry test_functions[] =
    {
       //参数 : 函数名 
       ZEND_FE(myecho, NULL)
       ZEND_FE_END
    };
    //注册到模块中
    zend_module_entry php_test_module_entry = {
    #if ZEND_MODULE_API_NO >= 20010901
       STANDARD_MODULE_HEADER,
    #endif
       "my test ext",//模块名
       test_functions,//函数列表
       NULL,
       NULL,
       NULL,
       NULL,
       PHP_MINFO(php_test),//模块介绍
    #if ZEND_MODULE_API_NO >= 20010901
       "1.6.0", //版本
    #endif
       STANDARD_MODULE_PROPERTIES
    };
    ```

    //函数实现,参考 3.函数部分

    ```
     PHP_FUNCTION(myecho){}
    ```

    用户函数调用: 参数: function\_table 函数表,全局从\&CG(function\_table)得到 对象,如果该函数是对象的方法,没有传NULL 函数名 返回值存储 zval指针, 参数数量 参数数组

    ```
     call_user_function(HashTable *function_table, zval **object_pp, zval *function_name, zval *retval_ptr, zend_uint param_count, zval *params[] TSRMLS_DC);
    ```
7.  类的实现

    结构:zend\_class\_entry 分两种 ZEND\_INTERNAL\_CLASS 和 ZEND\_USER\_CLASS

    ```
     //访问权限控制:ZEND_ACC_*系列宏
     //类方法定义
     //参数:类名 方法名 
     ZEND_METHOD( myclass , public_method )
     {
         //获取参数方法,参考:3.函数 部分
         if (zend_parse_method_parameters(ZEND_NUM_ARGS() TSRMLS_CC, getThis(), "s", &str,&strlen) == FAILURE) {
             RETURN_FALSE;
         }
         php_printf("hi function!");
     }
     //注册到函数列表
     static zend_function_entry myclass_method[]={
         //参数:类名 方法名 静态方法加 ZEND_ACC_STATIC
         ZEND_ME(myclass,    public_method,    NULL,    ZEND_ACC_PUBLIC)
         {NULL,NULL,NULL}
     };
     zend_class_entry ce;
     zend_class_entry *myclass_ce;
     //参数:初始结构 类名 方法列表
     INIT_CLASS_ENTRY(ce,"myclass",myclass_method);
     //此方法会根据ce拷贝生成一个zend_class_entry并挂载到类列表中
     zend_register_internal_class(&ce TSRMLS_CC);
     //添加类的属性
     //参数:类指针 属性名 属性名长度 属性访问权限 : zend_declare_property_* 系列函数
     zend_declare_property_null(myclass_ce, "public_var", strlen("public_var"), ZEND_ACC_PUBLIC TSRMLS_CC);
     //
     //定义类常量:
     //参数:类指针 属性名 属性名长度 属性访问权限 : zend_declare_class_constant* 系列函数
     zend_declare_class_constant_null(myclass_ce, "aaa", 3 TSRMLS_DC);
     //
     //读取类静态属性
         ZEND_API zval *zend_read_static_property(
             zend_class_entry *scope, 
             char *name, 
             int name_length, 
             //属性不存在是否抛出 notice
             zend_bool silent TSRMLS_DC
         );
     更新类静态属性
         ZEND_API int zend_update_static_property(
             zend_class_entry *scope, 
             char *name, 
             int name_length, 
             zval *value TSRMLS_DC
         );
     其他封装的快捷函数 zend_update_static_property_*
    ```
8.  接口 结构也是: zend\_class\_entry

    ```
     static zend_function_entry i_myinterface_method[]={
         //注意这里的null指的是arg_info,具体可以参考:参数 arg_info 的定义
         ZEND_ABSTRACT_ME(i_myinterface, hello, NULL) 
         ZEND_FE_END
     };
     //定义及初始化
     zend_class_entry ce;
     INIT_CLASS_ENTRY(ce, "i_myinterface", i_myinterface_method);
     i_myinterface_ce = zend_register_internal_interface(&ce TSRMLS_CC);
    ```
9.  继承及接口实现 1.实现接口: 写完类,接口后添加: 参数: 类指针 ,实现接口数量,接口指针...

    ```
         zend_class_implements(class_ce TSRMLS_CC,1,myinterface_ce);
    ```

    2.实现继承: 注册类时候使用: zend\_register\_internal\_class\_ex 参数: 类指针 父类指针 父类名

    ```
         myclass_ce = zend_register_internal_class_ex(&ce,parent_class_ce,"parent_class" TSRMLS_CC);
    ```
10. 对象 zend\_object\_value

    结构:

    ```
    typedef struct _zend_object_value {
        //unsigned int类型，EG(objects_store).object_buckets的索引
        zend_object_handle handle;
        zend_object_handlers *handlers;
    } zend_object_value;
    ```

    创建对象:

    ```
    zval *obj;
    MAKE_STD_ZVAL(obj);
    object_init_ex(obj, zend_class_entry 类名 );
    //object_init(obj) 空对象
    ```

    读取属性:

    ```
    ZEND_API zval *zend_read_property(
        zend_class_entry *scope, 
        zval *object, 
        char *name, 
        int name_length, 
        //属性不存在是否抛出 notice
        zend_bool silent TSRMLS_DC
    );
    ```

    更新属性:

    ```
    ZEND_API int zend_update_static_property(
        zend_class_entry *scope, 
        char *name, 
        int name_length, 
        zval *value TSRMLS_DC
    );
    ```

    其他封装的快捷函数 zend_update\_property_\* 调用方法:

    ```
    ZEND_API zval* zend_call_method(zval **object_pp, zend_class_entry *obj_ce, zend_function **fn_proxy, const char *function_name, int function_name_len, zval **retval_ptr_ptr, int param_count, zval* arg1, zval* arg2 TSRMLS_DC);
    zend_call_method_with_0_params(obj, obj_ce, fn_proxy, function_name, retval)
    zend_call_method_with_1_params(&this_zval,myclass_ce,NULL,"hello",NULL);
    ```
11. 常用结构及宏解释:

    ```
    PHP_FN(name) 函数名生成
    PHP_MN(name) 方法名生成
    ZEND_NAMED_FUNCTION(name) 生成指定名的函数 一般用
    PHP_FUNCTION(name) 函数声明
    ZEND_METHOD(classname, name) 类方法声明
    //
    zend_function_entry    数组用于声明模块或类的成员函数情况
    生成zend_function_entry的元素配套宏:
        //注册全局的函数
        //参数:函数名 参数
        PHP_FE(name, arg_info) 
        //注册类的函数
        //参数:类名 方法名 参数 访问权限控制
        PHP_ME(classname, name, arg_info, flags) 
        //映射一个函数为类的方法
        //参数:方法名 函数名 参数 访问权限控制 
        //用此宏须在函数体内进行类校验,zend_parse_method_parameters 
        //调用时会把当前对象作为第一个参数传递给函数
        PHP_ME_MAPPING(name, func_name, arg_types, flags)
        //注册类的抽象函数,类和接口都用这个
        //参数:类名 方法名 参数
        PHP_ABSTRACT_ME(classname, name, arg_info) 
        //结束标记
        PHP_FE_END
    ```
12. 参数 arg\_info 的定义

    ```
    //一般情况下不需要定义参数,除非你希望PHP般你检查参数正确性,比如接口方法
    //生成arg_info的宏,以arg_say_hello为例:
    //参数:zend_arg_info 名,参数至少前N个
    ZEND_BEGIN_ARG_INFO(arg_say_hello, 0)
    //参数:是否设置为引用,参数名
    //ZEND_ARG_* 系列函数还可以声明对象等
    ZEND_ARG_INFO(0, name)
    ZEND_END_ARG_INFO()
    //其他介绍
    //name    参数名称
    //name_len    参数名称字符串长度
    //class_name    当参数类型为类时，指定类名称
    //class_name_len    类名称字符串长度
    //array_type_hint    标识参数类型是否为数组
    //allow_null    是否允许设置为空
    //pass_by_ref    是否设置为引用，即使用&操作符
    //return_reference    标识函数将重写return_value_ptr，后面介绍函数返回值时再做介绍
    //required_num_args    设置函数被调用时，传递参数至少为前N个，当设置为-1时，必须传递所有参数
    ```
13. 目录结构

    ```
    ext/ 扩展
    ext/standard PHP内置函数
    Zend/ PHP核心
    Zend/zend_builtin_functions.c 核心函数 还有常用的辅助函数
    sapi/ sapi接口实现
    main/ 对外编写接口
    ```
14. 流程及扩展备注

    ```
    zend_module_entry php_test_module_entry = {
        STANDARD_MODULE_HEADER,
        "my test ext",                /* 模块名称 */
        test_functions,                /* 模块函数 */
        PHP_MINIT(test),            /* 模块启动 */
        PHP_MSHUTDOWN(test),        /* 模块关闭 */
        PHP_RINIT(test),            /* 请求开始 */
        PHP_RSHUTDOWN(test),        /* 请求完成 */
        PHP_MINFO(php_test),        /* 模块信息 */
        "1.6.0",                     /* 模块版本 */
        //还可以增加全局变量注册部分,参考:16.全局变量
        STANDARD_MODULE_PROPERTIES
    };
    ZEND_GET_MODULE(php_test);
    PHP_MINIT_FUNCTION(test)
    {
        //此阶段注册类等
        //注册INI
        //REGISTER_INI_ENTRIES();
        return SUCCESS;
    }
    PHP_RINIT_FUNCTION(test){
        return SUCCESS;
    }
    PHP_RSHUTDOWN_FUNCTION(test){
        return SUCCESS;
    }
    PHP_MSHUTDOWN_FUNCTION(test)
    {
        UNREGISTER_INI_ENTRIES();
        return SUCCESS;
    }
    ```
15. 配置设置与获取

    ```
    PHP_INI_BEGIN()
        PHP_INI_ENTRY("myini","bbb",PHP_INI_ALL, NULL)
    PHP_INI_END()
    //一般在 PHP_MINIT_FUNCTION 注册
    PHP_MINIT_FUNCTION(test)
    {
        //***
        REGISTER_INI_ENTRIES();
        //***
    }
    //获取配置:
    ///INI_* 系列宏
    ```
16. 线程安全的全局变量,并定义获取方法,类似于:EG(v) PG(v) SG(v) 等实现

    ```
    //test 一般为模块名,也可以用其他,用模块名号记忆
    //注意:全局变量不跟随请求,请求完成不会重置.
    ZEND_BEGIN_MODULE_GLOBALS(test)
        int flags;
    ZEND_END_MODULE_GLOBALS(test)
    #ifdef ZTS
    # define YOURGET(v) TSRMG(test_globals_id, zend_test_globals *, v)
    #else
    # define YOURGET(v) (test_globals.v)
    #endif
    //以上为声明部分
    //
    //以下为:初始化及注册到PHP内核中
    ZEND_DECLARE_MODULE_GLOBALS(test);
    //全局可用
    //ZEND_EXTERN_MODULE_GLOBALS(test);
    zend_module_entry php_test_module_entry = {
        STANDARD_MODULE_HEADER,
        "my test ext",
        test_functions,
        NULL,
        NULL,
        NULL,
        NULL,
        NULL,
        NO_VERSION_YET,//以下为开启全局变量时增加
        PHP_MODULE_GLOBALS(test),//作用:注册到全局变量中
        PHP_GINIT(test),//全局变量初始化时调用函数,一般为启动PHP时,可为NULL
        PHP_GSHUTDOWN(test),//全局变量销毁时调用函数,一般为关闭PHP时,可为NULL
        NULL,
        STANDARD_MODULE_PROPERTIES_EX //注意:多个EX
    };
    //全局变量注册的回调函数 宏
    static PHP_GINIT_FUNCTION(test)
    {
        test_globals->flags = 0;
    }
    static PHP_GSHUTDOWN_FUNCTION(test)
    {
        //其他操作,如释放申请的内存等
    }
    ```
17. gdb调试扩展

    PHP编译的时候加上 -–enable-debug 参数 修改config.m4

    ```
    if test -z "$PHP_DEBUG" ; then
       AC_ARG_ENABLE(debug,
               [--enable-debug compile with debugging system],
               [PHP_DEBUG=$enableval],[PHP_DEBUG=no]
       )
    fi
    ```

    ./configure 的时候加入 –enable-debug

    设置 core 的大小

    ```
    ulimit -c unlimited
    ```

    //以下假设php路径在系统的/usr/bin或/usr/local/bin中

    ```
    gdb php -c core[core为生成的core文件]
    ```

    常用调试命令:

    ```
    r 运行
    b 设置断点
    set args 设置参数
    n 运行下一步
    c 继续运行
    p 打印
    watch 当变量修改断点
    list  查看代码
    layout src 显示源码
    layout asm 显示汇编
    ```
