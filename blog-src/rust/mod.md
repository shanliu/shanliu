---
模块 mod
---


> 参考文件目录结构:

```bash
/src
/src/dome1
/src/dome1/mod.rs #文件夹mod入口约定文件,在这里包含子mod
/src/dome1/doma.rs #基于文件的mod,在mod.rs引入此文件
/src/domb.rs #基于文件的mod,在mod.rs引入此文件
/src/gdoma.rs #基于文件的mod
/src/lib.rs #当外部mod用,入口约定文件 生成文件名:lib+[包名:package.name]
/src/main.rs #存在此文件会编译可执行文件
```

> 引入mod和使用

```rust
//引入外部mod
//extern crate bba;
//导入宏,这样导入调用宏时不用写mod名
#[macro_use] extern crate bba;
//引入内部文件夹mod
mod dome1;
//引入内部文件mod
mod gdoma;
//在当前使用其他mod中定义宏,直接用不带mod名
#[macro_use] mod domb;
//使用非本文件[trait fn struct enum等]声明,方法如下:
//use bba::*;//外部库批量声明
use dome1::domep as bbb;//使用别名声明
use std::borrow::{BorrowMut, Borrow};//指定声明

//#[macro_use] 
// 1.用在外部crate时,该crate的下的导出宏不用改crate名使用
// 2.用在内部mod下时,可以直接使用该mod下定义的宏,也是唯一使用子mod下非#[macro_export]宏的方式
// 3.不同crate下 宏必须使用 #[macro_export] 导出,外部才能用
```

> 宏声明

```rust
//外部crate可用宏 #[macro_export] 等于直接把宏放置于crate根节点
#[macro_export]
macro_rules! test{()=>()}
//宏使用当前包函数等 $crate
pub fn increment(x: u32) -> u32 { x + 1 }
macro_rules! inc {
    ($x:expr) => ( $crate::increment($x) )
}
```

单文件多个mod,可见性控制 \[/src/gdoma.rs]

```rust
pub mod aaa{
    // trait 整体控制,不可到fn
    pub trait tdome1{//外部可见
        fn t1fn(&self);
    }
    trait tdome2{//外部不可见,对象试下以下方法包外部不可用
        fn t2fn(&self);
    }
    //属性单独控制可见性
    pub struct dome1{//字段 如果有一个非pub 将不可外部实例化
        pub a:i32,
        pub b:i32,
    }
    //方法单独控制可见性
    impl dome1{
        pub fn aaa(&self){//外部可访问
            self.bbb();
        }
        fn bbb(&self){//外部可访问
            self.bbb();
        }
    }
    impl tdome1 for dome1{//实现 pub trait tdome1 方法都外部可见
        fn t1fn(&self) {}
    }
    impl tdome2 for dome1{//实现 pub trait tdome1 方法外部都不可见
        fn t2fn(&self) {}
    }
    pub fn gdoma(){//外部可访问
        println!("gdoma");
    }
    pub mod bbb{
        pub fn gdoma(){
            self::gdomb();//self:: 自身mod
        }
        pub fn gdomb(){
            println!("gdomb");
        }
        pub mod ccc{
            pub fn gdoma(){
                super::super::gdoma();//super:: 上级mod
            }
        }
    }
    pub mod bbb1{
        pub fn gdoma(){
            crate::gdoma::aaa::gdoma();//crate:: 项目根,非当前文件
        }
    }
}
```
