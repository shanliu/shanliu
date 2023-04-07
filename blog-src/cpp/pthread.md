# 线程

```
##线程 pthread

#线程概念
    进程中的所有信息在线程中都是共享的
　　 编译参数 最后加上 -lpthread
    判断是否支持线程 sysconf(_SC_THREADS)
    线程ID是pthread_t 数据类型,各个平台的结构不相同
    每个线程都有errno副本

#基本线程函数
    #include <pthread.h>
    pthread_t pthread_self(void); 获取本身的线程ID
    int pthread_create(pthread_t *restrict tidp,
                       const pthread_attr_t *restrict attr,
                       void *(*start_rtn)(void),void *restrict arg);
        tidp 新线程的pthread_t的指针
        attr 新线程的属性 可以传null
        start_rtn 线程处理函数
        arg 传给线程的参数 可以传null
    int pthread_equal(pthread_t *pthread1,pthread_t *pthread2);
        判断是否是同一个线程
    void pthread_exit(void *rval_ptr); 线程终止
        *rval_ptr 线程停止时候的返回状态码
    int pthread_join(pthread_t thread,void **rval_ptr); 
        等待线程结束后执行,线程必须要处于非分离状态,分离线程使用pthread_detach
    int pthread_cancel(pthread_t tid);
        取消同一进程中的指定线程,单是提出请求,进程不一定会停止
    void pthread_cleanup_push(void (*rtn)(void *),void *arg);
        当线程以下情况时执行,可以添加多个:
            当线程执行 pthread_exit ,
            被其他线程调用pthread_cancel时,
            用非0执行pthread_cleanup_pop函数时
    void pthread_cleanup_pop(intg execute);
        清理pthread_cleanup_push添加的处理函数,当0时候不执行清理函数,多个调用多次清理
    int pthread_detach(pthread_t tid);
        分离线程

#线程同步
    1.互斥量
        防止相互加锁形成死锁,加锁后其他线程加锁将阻塞
        #include <pthread.h>
        int pthread_mutex_init(pthread_mutex_t *restrict muter,
                               const pthread_mutexattr_t *restrict attr);
            创建一个互斥量,也可以给pthread_mutex_t赋:PTHREAD_MUTEX_INITIALIZER;
                muter 是pthread_mutex_t类型的指针 ,通常为结构体的一部分
                attr 为该互斥量的一些属性
        int pthread_mutex_destroy(pthread_mutex_t * mutex);
            销毁一个互斥量
                muter 传给创建时候的pthread_mutex_t类型的指针
        int pthread_mutex_lock(pthread_mutex_t *muter);
             给一个互斥量加锁
        int pthread_mutex_trylock(pthread_mutex_t *muter);
            检查一个互斥量是否有枷锁,不会被阻塞.可以获得锁时候返回0 否则返回 EBUSY
        int pthread_mutex_unlock(pthread_mutex_t *muter);
            给一个互斥量解锁
        示例互斥量结构
            struct dome_mutex{
                int f_count;
                pthread_mutex_t f_lock;
            }
    2.读写锁(共享-独占锁)
        当一个线程加写锁,其他所有线程对该锁加锁都阻塞
        当一个线程加读锁,其他线程都可以继续加读锁,
        当加写锁被阻塞,直到所有的读锁解锁,后面的读锁也会阻塞,防止读锁占用时间过长
        #include <pthread.h>
        int pthread_rwlock_init(pthread_rwlock_t *restrict rwlock,
                                const pthread_rwlockattr_t *restrict attr);
            创建一个读写锁
        int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
            销毁一个读写锁
        int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
            加读 读写锁
        int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
            加写 读写锁
        int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
            解锁读或写的读写锁
        int phread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
            检查读写锁是否已经被锁为读锁,可以获得锁时候返回0 否则返回 EBUSY
        int phread_rwlock_trywrlock(prhread_rwlock_t *rwlock);
            检查读写锁是否已经被锁为写锁.可以获得锁时候返回0 否则返回 EBUSY
        示例读写锁结构
            struct dome_mutex{
                int f_count;
                pthread_rwlock_t f_lock;
            }
    3.条件变量
        条件变量需要配合互斥量使用
        #include <pthread.h>
        int pthread_cond_init (pthread_cond_t *restrict cond,
                               prhread_condattr_t *restrict attr);
            初始化条件变量,也可以给pthread_cond_t赋:PTHREAD_COND_INITIALIZER来初始化
        int pthread_cond_destory(pthread_coud_t *cond);
            释放条件变量
        int pthread_cond_wait(pthread_cond_t *restrict cond,
                              pthread_mutex_t *restrict mutex);
            等待条件为真返回变量
            cond 为pthread_cond_t条件变量
            mutex 为pthread_mutex_t互斥量
        int pthread_cond_timedwait(pthread_cond_t *restrict cond,
                                   pthread_mutex_t *restrict mutex,
                                   const struct timespec *restrict timeout);
            跟pthread_cond_wait功能相同,多一个超时,超时后返回ETIMEDOUT
            等待的时间结构,需要加当前时间
            struct timespec {
                time_t tv_sec;/*秒*/
                time_t tv_nsec;/*豪秒*/
            }
        int pthread_cond_signal(pthread_cond_t *cond);
            唤醒等待某个条件的某个线程,更改条件后调用 (POSIX规定跟pthread_cond_broadcast作用一样)
        int pthread_cond_broadcast(pthread_cond_t *cond);
            唤醒等待某个条件的所有线程,更改条件后调用

　　    备注:
            -> 得到互斥锁 (线程1)
　　　　　　　-> 设置条件变量,释放互斥锁 并等待触发 (线程1)
　　　　　　　-> 得到互斥锁 (线程2)
            -> 触发条件 (线程2)
            -> 释放互斥锁 (线程2)
            -> 触发,获得互斥锁 (线程1)
　　　　　　　-> 执行 (线程1)
            -> 释放互斥锁 (线程1)


##线程控制

#线程属性
    #include <pthread.h>
    线程属性结构pthread_attr_t 对程序是不透明的 具体参数参考:page 314 
    当不需要了解线程执行状态设置线程分离,线程执行完毕之后会自动回收分配的资源
    int pthread_attr_init(pthread_attr_t *attr);
        初始化一个线程属性
    int pthread_attr_destory(pthrad_attr_t *attr);
        销毁一个线程属性
    int pthread_attr_setdetachstate(const pthread_attr_t *restrict attr,
                                    int detachstate);
        设置线程分离分离状态 detachstate  可选值
            PTHREAD_CREATE_DETACHED 分离状态启动线程
            PTHREAD_CREATE_JOINABLE 正常状态启动进程
    int pthread_attr_getdetachstate(const pthread_attr_t *restrict attr,
                                    int *detachstate);
        获取指定线程属性的启动状态
    int pthread_attr_getstack(const pthrad_attr_t *restrict attr,
                              void **restrict stackaddr,
                              size_t *restrict stacksize);
        获取stackaddr线程属性
    int pthread_attr_setstack(cosnt pthread_attr_t (attr,
                              void *stackaddr,size_t *stacksize);
        设置stackaddr线程属性

#线程私有数据
    创建线程内全局变量

   int pthread_key_create(pthread_key_t *keyp,void(*destructor)(void*));
        创建进程的私有数据
            destructor 为当线程退出时候调用的函数指针,每个线程退出都调用
    int pthread_key_delete(pthread_key_t *keyp);
        删除进程的私有数据
    int pthread_setspecific(pthread_key_t *key,const void *pointer);
        设置KEY的数据
    void * pthread_getspecific(pthread_key_t *key);
        获取key的数据
```
