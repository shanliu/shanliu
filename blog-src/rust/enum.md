---
枚举 enum
---

> 枚举中闭包

```rust
 #[derive(Debug)]
    enum Ta{
        A(i32),//类型:参数为i32,返回自身的闭包 A(i32)->Ta
        B(i32)//类型:参数为i32,返回自身的闭包
    }
    //实现根据类型得到对应类型变量
    fn aaa<T:Fn(i32)->Ta>(a:T)->Ta{
        a(1)
    }
    let k=aaa(Ta::A);
    println!("{:?}",k);
    let k=aaa(Ta::B);
    println!("{:?}",k);
```

> enmu 示例

```rust
enum etest1{
    A,//跟C里面的枚举差不多
    B(f64),//待参数枚举,用于实现多类型变量
    C{a:i32}//结构枚举
}
//enum etest1{
//    A=2,//如果存在()或struct不可以有赋值,这将报错
//    B(f64)
//}
fn main() {
    let tvar1=etest1::A;
    if let etest1::A =tvar1 {//必须模式匹配,不能相等判断
        println!("match");
    }
    let tvar1=etest1::C{a:1};//带参数枚举
    if let etest1::C{a} =tvar1{
        println!("{}",a);
    }
    let tvar=etest1::B(0.1);//带参数枚举
    //枚举可以通过模式匹配得到枚举里面数据
    if let etest1::B(x) =tvar{
        println!("{}",x);
    }
}
```

> Option 类型

```rust
let b=(Some(1)).and_then(|x|Some('1'));//闭包返回更换OPTION类型,返回OPTION
let b=(Some(1)).and(Some('c'));//直接替换OPTION类型
let b=(Some(1)).map(|x|{'s'});//闭包返回更换OPTION类型,返回数据
let b=(Some(1)).replace(12);//替换掉值
let b=(Some(1)).filter(||true);//闭包返回真使用当前值,否则变为NONE
let b=(Some(1)).or(Some(1));//为NONE是使用指定值
let b=(Some(1)).or_else(||Some(1));//为NONE时闭包返回替换
let b=(Some(1)).ok_or(1);//转为RESULT,不存在时指定数据
let b=(Some(1)).ok_or_else(|x|1);//转为RESULT,不存在时指定闭包返回数据
let b=(Some(1)).take();//返回当前值并用默认值填充
let b=(Some(1)).map_or_else(||1,||2);//根据实际数据闭包返回解包
let b=(Some(1)).map_or(11,|x|{12});//默认值指定,数据根据实际数据闭包返回解包
let b=(Some(1)).expect("xxxx");//指定异常消息解包
let b=(Some(1)).unwrap();//按内部异常消息解包
let b=(Some(1)).xor(None);//异或处理 两个一样时候返回NONE
let b=(Some(1)).get_or_insert(1);//判断不存在时候使用指定值插入并解包
let b=(Some(1)).get_or_insert_with(||1);//判断不存在时候使用指定值插入闭包返回并解包
```

> Result<T,U>类型

```rust
    pub typeMYResult<T> = Result<T, String>;
    let b=(MYResult::Ok(1)).and_then(|x|MYResult::Ok('1'));//闭包返回更换OPTION类型,返回OPTION
    let b=(MYResult::Ok(1)).and(MYResult::Ok('c'));//直接替换OPTION类型
    let b=(MYResult::Ok(1)).map(|x|{'s'});//闭包返回更换OPTION类型,返回数据
    let b=(MYResult::Ok(1)).or(MYResult::Ok(1));//为NONE是使用指定值
    let b=(MYResult::Ok(1)).or_else(|e|MYResult::Err("cc".to_string()));//为NONE时闭包返回替换
    let c=(MYResult::Ok(1)).map_err(|x|"xxxx");//错误时回调替换
    let b=(Some(1)).filter(|e|true);//闭包返回真使用当前值
    let c=(MYResult::Ok(1)).err();//转为OPTION 正常为NONE错误为Some
    let c=(MYResult::Ok(1)).ok();//转为OPTION,正常为Some 错误为NONE
    let c=(MYResult::Ok(1)).is_err();//是否错误
    let c=(MYResult::Ok(1)).is_ok();//是否正常
    let c=(MYResult::Ok(1)).unwrap_err();//解包:解错误包,正常时异常
    let c=(MYResult::Ok(1)).unwrap();//解包:错误时异常
    let c=(MYResult::Ok(1)).unwrap_or(1);//解包:错误时用指定值代替
    let c=(MYResult::Ok(1)).unwrap_or_default();//解包:错误时用默认值
    let c=(MYResult::Ok(1)).unwrap_or_else(|x|11);//解包:错误时用指定闭包返回替换
    let b=(MYResult::Ok(1)).map_or_else(|e|1,|e|2);//解包:根据实际数据闭包返回解包
    let b=(MYResult::Ok(1)).map_or(11,|x|{12});//解包:默认值指定,数据根据实际数据闭包返回解包
```
