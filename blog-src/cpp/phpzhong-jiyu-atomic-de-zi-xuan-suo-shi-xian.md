# PHP中基于atomic的自旋锁实现

```c
typedef volatile unsigned long atomic_t;
#define atomic_cmp_set(a,b,c) __sync_bool_compare_and_swap(a,b,c)
static inline int fpm_spinlock(atomic_t *lock, int try_once) /* {{{ */
{
    if (try_once) {
        return atomic_cmp_set(lock, 0, 1) ? 1 : 0;
    }

    for (;;) {

        if (atomic_cmp_set(lock, 0, 1)) {
            break;
        }

        sched_yield();//出让CPU使用,转移线程到运行队列的尾部
    }

    return 1;
}
/* }}} */

#define fpm_unlock(lock) lock = 0
```

进程间共享内存

```c
void *fpm_shm_alloc(size_t size) /* {{{ */
{
    void *mem;

    mem = mmap(0, size, PROT_READ | PROT_WRITE, MAP_ANONYMOUS | MAP_SHARED, -1, 0);//进程间共享内存

#ifdef MAP_FAILED
    if (mem == MAP_FAILED) {
        return NULL;
    }
#endif

    if (!mem) {
        return NULL;
    }

    fpm_shm_size += size;
    return mem;
}
```

以下代码可以在多个进程间安全操作

```c
void* mem=fpm_shm_alloc(10);
atomic_t a;
fpm_spinlock(a,0);
//mem 处理mem内容
fpm_unlock(a)
```
