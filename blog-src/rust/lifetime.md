---
生命周期 lifetime
---

生命周期示例

```rust
use std::thread;
use std::borrow::Borrow;

//'a 为声明周期参数,必须先声明
fn aa1<'a>()->&'a str{
    //声明后生命周期参数可用
    let a:&'a str="ddd";//虽然声明a为'a 但a指向的内容为'static
    return a;//返回'static 引用合法
}
// fn aa4<'a>()->&'a i32{
//     let i=11;
//     let b:&'a i32=&i;
//     return b;//返回局部变量引用非法
// }
fn aa2<'a>()->&'a str{
    //声明后生命周期参数可用
    let mut a:String="ddd".to_string();
    //as_ref 会返回新建立的str的局部变量
    // 不能返回局部变量引用,所以会出错
    // return a.as_ref();
    return "dddd";//为'static >='a 所以可返回
}
// fn aa5()->&'static str{
//     let mut a:String="ddd".to_string();
//     //短生命周期数据不能转为长生命周期
//     let b:&'static str=a.as_ref();
//     return b;
// }
//'b:'a 为 存活时间 'b>='a
//主要作用为标识
fn aa3<'a,'b:'a>(str1:&'a str,str2:&'b str)->&'a str{
    //str1 str2 的生命周期告诉编译器传入变量那个会存活更久
    return str1;
}
//待生命周期参数结构
#[derive(Debug)]
struct at<'t>{
    a:&'t str
}
//'b:'a 为 存活时间 'b>='a
fn saa<'a,'b:'a>(s:i32,tmp:&'a at<'a>,tmp2:&'b at<'b>)
                 ->&'a at<'a>{//返回&'b at<'a>也可以,会以结构体'a为准
    if s>1 {
        return tmp;
    }else{
        return tmp2;
    }
}
//带生命周期的trait
trait Atr<'a>{}
impl <'a> Atr<'a> for at<'a>{
}
////返回如果是指定结构,需要执行生命周期 at<'a>
fn out<'a>(s:&'a str)->Box<at<'a>>{
    Box::new(at{
        a:s
    })
}
//返回如果是 trait 对象的话 需要使用 Atr<'a>+'a
fn foo<'a>(s:&'a str)->Box<Atr<'a>+'a>{//注意点
    Box::new(at{
        a:s
    })
}
//返回的值小于传入的任何一个,所以要返回'a
//带生命周期结构必须把生命周期带上
fn main() {
    //'static 全局声明周期
    let b:&'static str="dddd";
    let c=b.to_string();
    let c1:&str=b.as_ref();
    let b:&str=aa1();
    let b:&str=aa2();
    let b:&str=aa3("1","2");
    let b:String ="bbb".to_string();
    let va=&at{
        a:b.as_str()//可多次借用
    };
    let vc:&at;//老版本的有问题.
    let vb=&at{
        a:b.as_str()//可多次借用
    };
    vc=saa(1,va,vb);
    let c=out("tmp");
    println!("{:?},{:?}",vc,c);
    let raw="SSS".to_string();
    let b=raw.as_str();
    let b1=vec![1];
    let c=&b1[..];
    thread::spawn(move ||{
        //println!("{:?}",c);//&str 切片不可跨线程传
        //println!("{}",b);//&str 切片不可跨线程传
        println!("{}",raw);
    }).join().unwrap();
}
```

'static生命周期

```rust
use rand;
fn aa()->&'static i32{
    //静态编码,String 都是'static生命周期
    let str_literal: &'static i32 = &1;
    //例如这里直接返回 String, String 对象来说是'static生命周期的
    return str_literal;
}
fn rand_str_generator() -> &'static str {
    //不一定硬编码的才是'static 生命周期,如主动释放的也可以
    let rand_string = "sss".to_string();
    Box::leak(rand_string.into_boxed_str())
}
fn main() {
    let b=1;//所有权的变量生命周期也是'static
    let c=aa();
    println!("{}",c);
    let cc=rand_str_generator();
}
```

方法体self的生命周期

```rust

fn main() {
    //不可变借用.可以多次,所以'a内生命周期不需要释放
    struct abb<'a>(&'a i32);
    impl<'a> abb<'a>{
        fn a1(&'a  self) -> &'a i32 {
            return self.0;
        }
        fn a2(&'a  self) -> &'a i32 {
            return self.0;
        }
    }
    let mut a =abb(&1);
    a.a1();
    a.a2();

    //可变借用.可以同时只能借一次
    //声明了&'a mut self
    // 导致:调用aab1以'a生命周期调用a1方法 返回时应为'a的生命周期所以不释放自身的可变借用
    // 所以不能再调用第二次
    struct abb1<'a>(&'a i32);
    impl<'a> abb1<'a>{
        fn a1(&'a mut self) -> &'a i32 {
            return self.0;
        }
        fn a2(&'a mut self) -> &'a i32 {
            return self.0;
        }
    }
    let mut a =abb1(&1);
    a.a1();
    //a.a2();//再次调用会出错. 以为声明 &'a mut self 关系


    //可变借用.可以同时只能借一次
    //声明了&'_ mut self
    // 导致:调用aab1以'_生命周期调用a1方法
    //当返回时,'_的生命周期作用于当前函数,所以会释放self的可变借用
    // 所以可以再次调用
    struct abb2<'a>(&'a i32);
    impl<'a> abb2<'a>{
        //'_ 其实就是一个临时生命周期,默认系统会填充
        fn a1(&'_ mut self) -> &'a i32 {
            return self.0;
        }
        //'_ 其实就是一个临时生命周期,默认系统会填充
        fn a2(&'_ mut self) -> &'a i32 {
            return self.0;
        }
        //'b 不用系统填充的,自己定义
        fn a3<'b>(&'b mut self) -> &'a i32 {
            return self.0;
        }
    }
    let mut a =abb2(&1);
    a.a1();
    a.a2();
    a.a3();
}




```

泛型结构字段非字段生命周期

```rust
use std::marker;
struct Iter<'a, T: 'a> {
    ptr: *const T,
    end: *const T,
    _marker: marker::PhantomData<&'a T>,//特殊结构
}
//枚举使用 PhantomData
enum ValueBind<'q,'t:'q,T>where T:'q + Send
{
        a(&'t T),
        _tmp(std::convert::Infallible, std::marker::PhantomData<&'q T>),
}
let _=ValueBind::a(&1);
```

静态生命周期跟堆变量生命周期

```rust
struct Inspector<'a>(&'a u8);

struct Inspector1(&'static str);

impl<'a> Drop for Inspector1 {
    fn drop(&mut self) {
        println!("{}", self.0);
    }
}

fn main() {
    let (inspector, days); //生命周期 inspector>days
    days = Box::new(1);//当为Box因为在堆中
    inspector = Inspector(&days);//Inspector未实现 且Drop堆中正常编译
    // let days=1;
    // inspector = Inspector(&days);//days会比inspector先释放,无法编译

    let (inspector1, days1); //生命周期 inspector>days
    days1 = "ssss";
    inspector1 = Inspector1(&days1);//当days1为static时正常编译
    //如果Inspector1的str非'static 无法编译

}
```

未在结构的生命周期

```rust
trait Ba<'c,T>{
    fn a(a: &'c T)->&'c T{
        return a;
    }
}
struct TT{}
impl<'c,T> Ba<'c,T> for TT {}
struct Ptype<'c,T>//类型 生命周期或泛型未使用在字段时 PhantomData 关联两者关系
    where
        T:Ba<'c,i32>,
{
    a:T,
    _marker:std::marker::PhantomData<&'c T>
}
fn main() {
    let t=TT{};
    let t1=Ptype{
        a:t,
        _marker: Default::default()
    };
}
```

> 闭包指定生命周期

```rust

let identity: &dyn for<'a> Fn(&'a i32) -> &'a i32 = &|x: &i32| x;
let b=identity(&1);
println!("{}",b as &i32);
```
