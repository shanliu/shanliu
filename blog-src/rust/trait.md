---
约束 Trait
---


> 基本示例

```rust
trait Atest{
    fn aa0(self);
   // fn aa1(mut self);//错误:trait不可以存在可变实体变量
   // fn aab(mut a: Aa);//错误:trait不可以存在可变实体变量
    fn aa2(&self);
    fn aa3(&mut self);
    //fn st2()->impl Debug; //错误:impl trait 只能对fn跟实体方法返回 trait 用Box<dyn trait>返回
}
#[derive(Debug)]
enum etest1{
    A(i32),
    B(f64)
}
trait a<T>{

}
trait b<T>:a<T>{//带泛型继承

}
trait  ttest1{
	//方法名不可相同.跟CPP不一样,同名方法要多态用泛型实现
    fn fn1(&self)->i32;//引用,可多次
    fn fn2(self)->i32;//只执行一次
}
trait  ttest2:ttest1{//继承
    fn fn3(&self)->i32{
		//注意: self为引用 &self 为引用的引用
        return 1;
    }
}
impl ttest1 for etest1{
    fn fn1(&self) -> i32 {
        if let etest1::A(x) =*self{//模式匹配得到数据
            return x;
        }
        return 0;
    }
    fn fn2(self) -> i32 {
        return 1;
    }
}
impl etest1{
    fn fn4(){}
}
impl ttest2 for etest1{}//继承实现,增加功能
fn main() {
    let s=etest1::A(1);
    s.fn1();
    s.fn3();
    s.fn2();
    etest1::fn4();
}
```

> trait 不的函数不支持返回 impl Trait 类型,用Box\<dyn \*>返回

```rust
use std::future::Future;
use std::pin::Pin;
struct Sk<'c>{
    a:&'c str
}
struct Sa{}
trait Ta{
    //trait 的方法不支持 impl 返回
    fn aa<'c>(&self,sss:&'c str)->Pin<Box<dyn Future<Output=Sk<'c>>+'c>>;
}
impl Ta for Sa{
    fn aa<'c>(&self,sss:&'c str) ->Pin<Box<dyn Future<Output=Sk<'c>>+'c>>{
        let a=Sk{
            a:sss
        };
        Box::pin(async move {a})
    }
}
#[tokio::main]
async fn main()  {
    let sa=Sa{};
    let b="ddd".to_string();
    let b=b.as_str();
    let m=sa.aa(b).await.a;
    println!("{}",m);
}

```

> 带限制泛型实现trait

```rust
trait aa{
    fn out(&self)->i32{
        return 1;
    }
}
trait tl{}
impl<T:tl> aa for T{}//给实现了tl的trait对象实现aa trait 注意:如果没限制会加到所有类型上

struct dome1{}
impl tl for dome1{}//dome1实现了 tl 所以实现aa

fn main() {
    (dome1{}).out();
}
```

> 全局泛型实现trait

```rust
trait aa{
    fn out(&self)->i32{
        println!("ddd");
        return 1;
    }
}
impl<T> aa for T{}//等于给所有类型加了aa trait的out方法
fn main() {
    "aa".out();
    1.out();
}
```

> 全局泛型多个相同方法trait实现

```rust
trait aa{
    fn out(&self)->i32{
        println!("ddd");
        return 1;
    }
}
trait bb{
    fn out(&self)->i32{
        println!("ddd");
        return 1;
    }
}
impl<T> aa for T{}//等于给所有类型加了aa trait的out方法
impl<T> bb for T{}//等于给所有类型加了bb trait的out方法
fn main() {
    bb::out(&"aa");//多次实现,必须手动指定调用那个trait方法
}
```

> 全局泛型跟结构实现方法同名

```rust
trait aa{
    fn out(&self){
        println!("1");
    }
}
impl<T> aa for T{}//等于给所有类型加了aa trait的out方法
struct dome1{}
impl dome1{
    fn out(&self){//跟全局泛型有同名out方法,会覆盖全局泛型方法
        println!("2");
    }
}
fn main() {
    (dome1{}).out();
    "22".out();
}
```

> dyn trait 对象和结构对象转换

```rust
struct aa{}
impl aa{
    fn aaa(&self){
        println!("ddd");
    }
}
trait bb{}
impl bb for aa{}
fn main() {
    let m=aa{};
    let b= &m as &bb;//转为 dyn trait 对象
    let c:&aa;
    unsafe { c=&*(b as *const dyn bb as *const aa); }//dyn trait对象转为指定类型对象,自行保证类型安全
    c.aaa();//转为指定类型对象后调用该类型方法
}
```

> 判断是否某类型 && 泛型直接调用

```rust
println!("{}",(&"dddd" as &Any).is::<i32>());//判断是否实现某类型
struct Atest{}
struct Atest1{}
println!("{}",(&Atest{} as &Any).is::<Atest1>());//判断自定义类型
//注意 : 无法实现判断是否实现某 trait 
```

> 动态triat对象返回 Box\<dyn triat>  \[动态分发]

```rust
trait ttest1{
    fn out(&self);
}
struct test1{}
impl ttest1 for test1{
    fn out(&self){
        println!("test fn1");
    }
}
struct test2{}
impl ttest1 for test2{
    fn out(&self){
        println!("test fn2");
    }
}
fn out(i:i32)->Box<dyn ttest1>{//返回动态对象方法 Box<dyn trait>
    if(i>0) {
        return Box::new(test1 {});
    }else{
        return Box::new(test2 {});
    }
}
fn main() {
    let b=out(1);//返回动态对象 可以用any判断是什么对象
    b.out();
    let b=out(0);//返回动态对象
    b.out();
}
```

> `impl trait` 对象返回 `静态分发,闭包就是其中一个实现`

```rust
use std::iter;
use std::vec::IntoIter;
// This is the exact same function, but its return type uses `impl Trait`.
// Look how much simpler it is!
fn combine_vecs(
    v: Vec<i32>,
    u: Vec<i32>,
) -> impl Iterator<Item=i32> {
    v.into_iter().chain(u.into_iter()).cycle()
}

fn main() {
    let v1 = vec![1, 2, 3];
    let v2 = vec![4, 5];
    let mut v3 = combine_vecs(v1, v2);
    assert_eq!(Some(1), v3.next());
    assert_eq!(Some(2), v3.next());
    assert_eq!(Some(3), v3.next());
    assert_eq!(Some(4), v3.next());
    assert_eq!(Some(5), v3.next());
    println!("all done");
}
```

> 通过 trait 实现类型约束

```rust
trait typeLimitTrait{}//自定义约束用trait
impl typeLimitTrait for i32 {}//实现某类型
impl typeLimitTrait for f32 {}//现某类型
fn out1<T:typeLimitTrait>(a:T)->T{//限定泛型
    return a;
}
```

> 模拟多态

```rust
trait MT1<Ttest>{
    fn t1(a:Ttest){}
}
struct ST1{}
impl MT1<i32> for ST1{
    fn t1(a:i32){}
}
impl MT1<i8> for ST1{
    fn t1(a:i8){}
}
```

> 同类型操作符重载

```rust
struct Foo {
    value:i32
}
impl Add for Foo{
    type Output = Self;
    fn add(mut self, rhs: Self) -> Self::Output {
        self.value=self.value+rhs.value;
        self
    }
}
fn main() {
    let mut x = Foo { value: 10 };
    let mut b = Foo { value: 10 };
    let c=x+b;
}
```

> 不同类型操作符重载和泛型限定

```rust
struct Foo {
    value:i32
}
impl Add<i32> for Foo{
    type Output = i32;
    fn add(mut self, rhs: i32) -> Self::Output {
        return self.value+rhs;
    }
}
//注意!!!! 限定 Add<i32,Output=i32> 而不是 Add<i32>
//如果限定为 Add<i32> 则输出为 Add<i32>::Output 而不是 i32 因为 Add<i32>未指定 Output
fn add<T:Add<i32,Output=i32>>(a:T)->i32{
    let b=a+1;//可以+i32 是因为实现了 Add<i32> 
    // 输出 b 为i32 是因为: 
    // Add<i32,Output=i32> 让输入参数的返回为i32 
    // impl Add<i32> for Foo 的 type Output = i32; // 让a+1 返回为i32
    // 以上都是必须的
    return b;
}
fn main() {
    println!("{}",add(Foo { value: 10 }));
    //Foo { value: 10 }+10 返回 i32 是因为 impl Add<i32> for Foo 的 type Output = i32;
    //注意跟 add 函数中的区别
    println!("{}",Foo { value: 10 }+10);
}
```

> 带泛型triat实现

```rust

//trait 泛型 T1=i8 表示默认泛型
trait Dome1<T,T1=i8>{
    //如果用作泛型限制,OUT1跟OUT2可以指定
    // 不指定,调动完test1 的类型为 Dome1<T,T1=i8>::OUT1
    // 指定 T:Dome1<T,T1=i8,OUT1=i8,OUT2=i16> 调动完test1 的类型为 i8
    type OUT1;//内部泛型,实现时不写在外部
    type OUT2;//注意:使用不能是 OUT2 必须为 Self:: 方式 例如 Self::OUT2
    fn test1(&self,a:T,b:T1)->Self::OUT1;//Self 表示当前类型
    fn test2(&self,a:T,b:T1)->Self::OUT2;
}
#[derive(Debug)]
enum MYENUM{
    a=1
}
//全部指定类型trait实现
//指定T=i32,T1=i8实现
impl Dome1<i32,i8> for MYENUM{
    type OUT1=i32;
    type OUT2=i8;
    fn test1(&self,a: i32, b: i8) -> Self::OUT1 {//Self 表示当前类型 即:MYENUM
        return a + b as i32;
    }
    fn test2(&self,a: i32, b: i8) -> Self::OUT2 {
        //&self 表示当前示例引用 可以 . 方式调用
        return a as i8 + b ;
    }
}
#[derive(Debug)]
enum MYENUM1{
    a=1
}
//使用Dome1默认的泛型trait实现
impl Dome1<i32> for MYENUM1{
    type OUT1=i32;
    type OUT2=i8;
    fn test1(&self,a: i32, b: i8) -> Self::OUT1 {
        return a + b as i32;
    }
    fn test2(&self,a: i32, b: i8) -> Self::OUT2 {
        return a as i8 + b ;
    }
}
#[derive(Debug)]
enum MYENUM2{
    a=1
}
//实现时使用泛型方式实现泛型trait在指定类型
impl<T> Dome1<T> for MYENUM2{
    type OUT1 = T;
    type OUT2 = i8;
    fn test1(&self, a: T, b: i8)->Self::OUT1{
        return a;
    }
    fn test2(&self, a: T, b: i8) ->Self::OUT2 {
        return b;
    }
}
fn test1<T>(a:T)->i32
    //where方式
    //注意:用作限定时 内部OUT*不是必须写的
    where T:Dome1<i32,i8,OUT1=i32>+Debug{
    return a.test1(1,2);
}
fn main() {
    let a=MYENUM1::a;
    a.test1(1,2);
    let b=MYENUM::a;
    test1(b);
    let c=MYENUM2::a;
    c.test1(1, 2);//泛型自动推导
}
```

> 指定某泛型类型调用

```rust
trait Ltrait{
    fn test(&self);
}
impl Ltrait for i8{
    fn test(&self) {
        println!("i8");
    }
}
impl Ltrait for i32{
    fn test(&self) {
        println!("i32");
    }
}
fn bstest<T:Ltrait>(a:T){
    //限制 a的可传入类型
    a.test();
}
fn main() {
    1i32.test();//调用已被赋值到类型方法
    1i8.test();//调用已被赋值到类型方法
    bstest::<i8>(1);//指定 Ltrait for i8 实现调用
    // 全局泛型实现的指定类型调用
    println!("{}",<f32>::from(1.0));//指定数据类型泛型调用
}
```

> 往dyn trait对象增加方法

```rust
#[test]
fn test1(){
    trait Aa{}
    struct Bb{}
    impl Bb{
        fn out(&self){
            println!("aa");
        }
    }
    impl Aa for Bb{}
    impl dyn Aa{
        //改动态对象增加方法
        fn out(&self){
            println!("bb");
        }
    }
    let b=Bb{};
    b.out();//调动到Bb的out方法
    (&b as &dyn Aa).out();//调用到 dyn Aa的方法
}

#[test]
fn test2(){
    trait Aa{}
    trait Ac{
        fn out(&self);
    }
    struct Bb{}
    impl Aa for Bb{}
    impl Ac for dyn Aa{
        //改动态对象增加triat实现
        //实现Aa的方法存在out方法名时需指定triat调用
        fn out(&self){
            println!("bb1");
        }
    }
    let b=Bb{};
    (&b as &dyn Aa).out();
}

#[test]
fn test3(){
    trait Aa{
        fn out(&self);
    }
    trait Ac{
        fn out(&self);
    }
    struct Ba{}
    impl Aa for Ba{
        //实现 trait
        fn out(&self){
            println!("aa");
        }
    }
    impl Ac for dyn Aa{
        //动态对象重新实现某方法,需要指定该triat调用
        //跟实现多个triat存在重名方法一致.
        fn out(&self){
            Aa::out(self);
            println!("bb");
        }
    }
    let b=(&Ba{} as &dyn Aa);
    Aa::out(b);
    b.out();//等于调用 Aa::out(b);
    Ac::out(b);//修改动态对象的指定方法
}
```

>  PhantomData 使用

```
fn main() {
    struct Aa<T>{
        a:i32,
        _marker:std::marker::PhantomData<T>
    }
    Aa{
        a: 0,
        _marker:std::marker::PhantomData::default()
    }
    as Aa<i32>;//运行时泛型具体到某类型 PhantomData 类型未知,所以运行时不指定将出错
}
```

> 通过泛型实现类型一级继承

```rust
struct Ptype<SUBTYPE>{
    _marker:std::marker::PhantomData<SUBTYPE>
}
impl<SUBTYPE> Ptype<SUBTYPE>{
    fn new()->Self{
        Ptype{
            _marker:std::marker::PhantomData::default()
        }
    }
}
struct Sub1Type{}
impl Ptype<Sub1Type>{
    fn aa(&self)->i32{
        return 1;
    }
}
struct Sub2Type{}
impl Ptype<Sub2Type>{
    fn bb(&self)->i32{
        return 2;
    }
}
fn main() {
    print!("{}",(Ptype::new() as Ptype<Sub1Type>).aa());
    print!("{}",(Ptype::new() as Ptype<Sub2Type>).bb());
}
#[test]
fn test1(){
    struct A<SUBTYPE>{
        _marker:std::marker::PhantomData<SUBTYPE>
    }
    let t1:A<i32>=A{_marker:Default::default()};#运行时得到类型
}
//struct MYTYPE<SUBTYPE>
// SUBTYPE:LIMIT_TRAIT
//     实现 LIMIT_TRAIT 一批子类型增加方法实现
//     SUBTYPE指定类型实现

```

> 无指定类型实现trait调用问题

```rust
trait aaa1{
    fn out(){}
}
trait aaa2{
    fn out(&self){
    }
}
impl<T> aaa1 for T{}
impl<T> aaa2 for T{}
fn main() {
    //aaa1::out(); //cannot infer type
    aaa2::out(&1);//能推导出当前附属到的类型
}
```

> 复合类型中泛型限定方法增加

```rust
trait Mlimit{}
//impl<T> Mlimit for T{} //最顶层泛型 全局泛型实现
struct Mytest<T1> {a:T1}
//impl<T1:Mlimit> Mlimit for Mytest<T1> {} //类型实现 trait
//impl<T3> Mytest<T3>{}//类型增加方法
impl<T3:Mlimit> Mytest<T3>//给类型内部泛型限定下增加该类型方法
{
    fn fn1(self) {}
}
impl Mlimit for i32{}
(Mytest{a:1}).fn1();
```

> 泛型使用时不能做特化处理

```rust

trait IDB {}
struct IMY1 {}
impl IDB for IMY1 {}
struct IMY2 {}
impl IDB for IMY2 {}
fn bind1<T: IDB>(_: T) {}
struct F1<DB:IDB>{_tmp:DB}
impl <DB:IDB>F1<DB>{
    fn _myexec(_:DB)  {
        //泛型不能特化处理
        //bind1::<DB>(IMY1{});
    }
}
impl F1<IMY1>{
    fn myexec()  {
        bind1(IMY1{});
    }
}
impl F1<IMY2>{
    fn myexec()  {
        bind1(IMY2{});
    }
}
fn main() {
    F1::<IMY1>::myexec();
    F1::<IMY2>::myexec();
}
```

> 泛型约束示例

```rust
trait IDB {}
struct IMY{}
impl IDB for IMY{}
trait Encode<DB:IDB> {}
struct Query<DB:IDB>{
    _a:DB
}
impl <DB: IDB> Query<DB> {
    fn bind1<T:Encode<DB> >(&self, _: T)  {}
}
impl Encode<IMY> for i32 {}
fn main() {
    fn myexec<DB:IDB>(pool:DB) 
    where i32:Encode<DB> //加约束实现泛型替换关联
    {
        let res =Query{_a:pool};
        res.bind1(1);
    }
    myexec::<IMY>(IMY{});
}rs
```
