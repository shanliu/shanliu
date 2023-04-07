---
 异步
---

# async

> 执行器过程:

1. 由IO复用或定时阻塞的`轮训循环`
2.  在循环内调用 future 的poll方法\[传入触发完成的回调函数的上下文Context]

    ```
    //`轮训循环` 循环内的伪代码:
    let task = Arc::new(Task {});//TASK 必须实现 ArcWake
    let waker = waker_ref(&task);
    let context = &mut Context::from_waker(&*waker);//传给 poll
    future.boxed().as_mut().poll(context);
    //同时将waker 挂载到future 属性上
    ```

    ```
    //完成时从futrue的属性上获取到waker并调用 waker.wake() 后会进入以下函数
    impl ArcWake for Task {
     fn wake_by_ref(arc_self: &Arc<Self>) {
         //完成触发后会进入这里
         //这里触发来源:自定义future的poll函数内的回调挂载的:context.waker().clone() 同时挂载到future属性
         //这里触发时,`轮训循环` 一般进入等待状态
         //这里进行某些操作后,`轮训循环`得于继续运行并调用poll
     }
    }
    ```
3. 全部完成时结束循环

> future 分类

1.  async 代码转换实现future

    ```
    let a=async{//跟闭包类似,可以return返回
     let b=async{};
     b.await;
     //遇到 await 时,会调用对应future的poll函数
     //如果函数返回 Pending,会一直返回到 最顶层 `轮训循环`
     //因为自定义的poll函数内不会有 await 语法,
     //所以返回 Pending时可以保证在一直返回到最顶层
     //如果函数返回Ready,继续async块的代码执行
     //二次进入：（resume）时：  
     //先到async块，然后跳转到跳出时的await位置 
     //在根据get_context(cx:ResumeTy) 得到cx并从中获取上次跳出时的 future
     //再次调用该future的poll方法
    };
    ```

    ```
    //async块的转换的 future 是包含生成器(gen) 
    //async块的转换的 future poll 一般会多于2次进入
    //自定义的future的一般就2次，一次等待一次完成
    //async块的转换的 future poll 函数内容如下：
    //由生成器的 gen.resume 实现运行由poll的内容到 async{这里的内容} 的跳转运行
    //async块的转换的 future poll 本身不会有暂停或阻塞的情况出现
    let gen = unsafe { Pin::map_unchecked_mut(self, |s| &mut s.0) };
    match gen.resume(ResumeTy(NonNull::from(cx).cast::<Context<'static>>())) {
     GeneratorState::Yielded(()) => Poll::Pending,//保证返回到`最顶层循环`代码
     GeneratorState::Complete(x) => Poll::Ready(x),//完成时返回
    }
    ```
2.  自定义 future 实现 impl Future&#x20;

    ```
    impl Future for TimerFuture {
     type Output = ();
     fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
         //自定义future一般只会进入2次
         //第1次
         //如果未完成,返回:Poll::Pending 这里一般是需要暂停或阻塞情况
         //同时挂载回调函数 = Some(cx.waker().clone());//这个挂载可在其他线程或内部一些异步IO库中
         //第2次
         //完成时poll返回:Poll::Ready(())
         //await一定是在async的异步代码中,所以这里不会有await情况
         //除非实现类似async块的转换的 future 的结构(gen)否则不会大于2次进入poll函数
     }
    }
    ```

> future的内部转换

```
async => future{
    //异步函数的过程内容转为POLL过程内容，类似闭包
    //局部变量转为属性[有局部变量引用时候存在问题,所以有pin]
    //函数调用转为一个struct[类似闭包转换]挂到属性.
}
//Future转换过程
Future{
    state//状态机,跟踪进度用 跳转到那个await
    a//局部变量转为普通变量
    struct b//内部函数调用转为结构
    Future c//遇到 await 转为Future结构挂载到属性
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>{
       //async 代码转换实现future 内部实现 代码省略
    }
}
```

async块Result类型指定返回

```rust
let a=async{return Result::<i32,i32>::Ok(2)};//<>操作符
```

Pin加Unpin实现:

```rust
fn execute_unpin_future(x: impl Future<Output = ()> + Unpin) {}
let fut = async {};
let fut = Box::pin(fut);//跟使用futrue的boxed方法一样
execute_unpin_future(fut); // OK
let fut = async {};
pin_utils::pin_mut!(fut);//跟上面Box::pin(fut)类似
execute_unpin_future(fut); // OK
//实现了Unpin表示可以安全移动 一般无内部局部变量引用都实现了Unpin 如常用的基本类型

```

多个async同步

```rust
struct Book;
struct Music;
async fn get_book() -> Book { Book }
async fn get_music() -> Music { Music }


mod join {
    use super::*;
    use futures::join;
    //两个future完成
    pub(crate) async fn get_book_and_music() -> (Book, Music) {
        let book_fut = get_book();
        let music_fut = get_music();
        join!(book_fut, music_fut)
    }
    #[test]
    fn get_book_and_music_test() {
        futures::executor::block_on(get_book_and_music());
    }
}

mod try_join {
    use super::{Book, Music};
    use futures::try_join;
    //两个future完成,其中一个返回失败立即返回
    async fn get_book() -> Result<Book, String> { /* ... */ Ok(Book) }
    async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }
    async fn get_book_and_music() -> Result<(Book, Music), String> {
        let book_fut = get_book();
        let music_fut = get_music();
        try_join!(book_fut, music_fut)
    }
    #[test]
    fn get_book_and_music_test() {
        futures::executor::block_on(get_book_and_music());
    }
}

mod mismatched_err {
    use super::{Book, Music};
    use futures::{
        future::TryFutureExt,
        try_join,
    };
    async fn get_book() -> Result<Book, ()> { /* ... */ Ok(Book) }
    async fn get_music() -> Result<Music, String> { /* ... */ Ok(Music) }
    async fn get_book_and_music() -> Result<(Book, Music), String> {
        let book_fut = get_book().map_err(|()| "Unable to get book".to_string());
        let music_fut = get_music();
        try_join!(book_fut, music_fut)
    }
    #[test]
    fn get_book_and_music_test() {
        futures::executor::block_on(get_book_and_music());
    }
}

mod example {
    // ANCHOR: example
    use futures::{
        future::FutureExt, // for `.fuse()`
        pin_mut,
        select,
    };
    async fn task_one() { /* ... */ }
    async fn task_two() { /* ... */ }
    //select等待
    async fn race_tasks() {
        let t1 = task_one().fuse();
        let t2 = task_two().fuse();
        pin_mut!(t1, t2);
        select! {
            //返回=future=>操作
            () = t1 => println!("task one completed first"),
            () = t2 => println!("task two completed first"),
        }
    }
    #[test]
    fn race_tasks_test() {
        futures::executor::block_on(race_tasks());
    }
}

mod default_and_complete {
    use futures::{future, select};
    async fn count() {
        let mut a_fut = future::ready(4);
        let mut b_fut = future::ready(6);
        let mut total = 0;
        loop {
            select! {
                a = a_fut => total += a,
                b = b_fut => total += b,
                complete => break,//全部完成
                //default 在这里不会生效,因为 future::ready 是终止的
                default => unreachable!(), // future都是未完成且非终止的进这里,需要通过自己的future.await 处理
            };
        }
        assert_eq!(total, 10);
    }
    #[test]
    fn run_count() {
        futures::executor::block_on(count());
    }
}


mod channels {
    use {
        futures::{
            channel::mpsc,
            prelude::*,
        },
    };
    async fn send_recv() {
        const BUFFER_SIZE: usize = 10;
        let (mut tx, mut rx) = mpsc::channel::<i32>(BUFFER_SIZE);
        tx.send(1).await.unwrap();
        tx.send(2).await.unwrap();
        drop(tx);
        assert_eq!(Some(1), rx.next().await);
        assert_eq!(Some(2), rx.next().await);
        assert_eq!(None, rx.next().await);
    }
    #[test]
    fn run_send_recv() { futures::executor::block_on(send_recv()) }
}

```

多个future 的遍历

```rust
use async_std::task;

use futures::{
    stream::{Stream, FusedStream},
    select,
    SinkExt,
    StreamExt,
};
use async_std::sync::Arc;
use std::time::Duration;

async fn add_two_streams(
    mut s1: impl Stream<Item = i32> + FusedStream + Unpin,
    mut s2: impl Stream<Item = i32> + FusedStream + Unpin
) -> i32 {
    let mut total = 0;
    loop {//select 将多个future合并成一个,生成方式如下:通过实现 FusedFuture::is_terminated 判断时完成处理future
		//创建合并future的闭包,把上面的多个future放入一个闭包,由闭包内调用future的poll方法
		//在根据是否进行default定义进行以下两种处理
		//1.未指定:default,把闭包通过 futures::future::poll_fn 转为 future 后进行 future.await ,承接到外部运行时
		//2.指定:default,直接调用创建的闭包,如果闭包未返回完成,承接到定义的default里处理
        let item = select! {//遍历future
            x = s1.next() => x,
            x = s2.next() => x,
            // default =>{
			//	当`未终止`的所有future`都是`Pending 时这里
			//  当需要`自定义`都未准备完成时的处理时才定义这里
            //  return myfuture{}.await;//例如这里的myfuture.await 这样承接到外部运行时
			//  Some(1) //如果直接返回值,外部用loop循环时会浪费大量CPU.所以一般要定义里面必须会有await
            // },
            complete => break,//future都完成进这
        };
        println!("select:{:?}",item);
        if let Some(next_num) = item {
            total += next_num;
        }
    }
    total
}
fn main() {
    task::block_on(async{
        let  (mut send, mut recv1)= futures::channel::mpsc::channel::<i32>(10);
        send.send(11).await.unwrap();
        send.send(20).await.unwrap();
        drop(send);
        let  (mut send, mut recv2)= futures::channel::mpsc::channel::<i32>(10);
        std::thread::spawn(move ||{
            std::thread::sleep(Duration::from_secs(10));
            task::block_on(async{
                send.send(21).await.unwrap();
                send.send(22).await.unwrap();
                drop(send);
            });
        });
        let t=add_two_streams(recv1,recv2);
        let t1=t.await;
        println!("{}",t1);
    });
}
```

异步递归调用

```rust
use futures::future::{BoxFuture, FutureExt};
fn recursive(mut a:i32) -> BoxFuture<'static,i32> {
    async move {
        a=a+1;
        if a<100 {
            return  recursive(a).await;
        }
        a
    }.boxed()
}
recursive(1).await;
```
