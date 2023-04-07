---
字符数字转换
---


```rust
use std::error::Error;
use std::convert::{TryFrom, TryInto};

fn main() {
    //数字转字符串
    let num=1.0/3.0;//浮点数
    let inum=num as i32;//下取整
    let anum=(num as f64).ceil();//向上取整
    let str=num.to_string();
    println!("{}",format!("num:{:.2}str:{}inum:{}anum:{}",num,str,inum,anum));
    let num=1u8;//U8定义
    let str=num.to_string();
    println!("{}",format!("num:{}str:{}",num,str));
    let num=b'c';//U8定义
    let str=num.to_string();
    println!("{}",format!("num:{}str:{}",num,str));
    let num=1i32;//8位整数
    let str=num.to_string();
    println!("{}",format!("num:{}str:{}",num,str));
    let num=1.01f32;//32位浮点
    let str=num.to_string();
    println!("{}",format!("num:{:.2}str:{}",num,str));
    //char 字符类型 u8 i* 都是整数
    let a= '1' as i32 as u8 as char;
    let b=49i32 as i8 as u8 as char;
    println!("{}{}",a,b);
    //字符串转数字
    let num="1".parse::<i32>().unwrap();
    println!("{}",num);
    let num="??".parse::<i32>();
    if num.is_err() {
        println!("{}",num.unwrap_err().to_string());
    }
    let num="1.11111111".parse::<f32>().unwrap();
    println!("{}",num);
    //自定义类型转换
    #[derive(Debug)]
    struct mynum{
        a:i32,
        b:i32
    }
    impl From<String> for mynum{
        fn from(str: String) -> Self {
            let split=str.split(",");
            let n=split.collect::<Vec<_>>();
            if(n.len()<2){
                return mynum{
                    a:0,
                    b:0
                };
            }
            let a:i32=n[0].parse::<i32>().unwrap();
            let b:i32=n[1].parse::<i32>().unwrap();
            return mynum{
                a:a,
                b:b
            }
        }
    }
    #[derive(Debug)]
    struct mynum1{
        a:i32,
        b:i32
    }
    impl TryFrom<String> for mynum1{
        type Error = &'static str;
        fn try_from(str: String) -> Result<Self, Self::Error> {
            let split=str.split(",");
            let n=split.collect::<Vec<_>>();
            if(n.len()<2){
                return Err("bad");
            }
            let a:i32=n[0].parse::<i32>().or_else(|e|Err("a bad"))?;
            let b:i32=n[1].parse::<i32>().or_else(|e|Err("b bad"))?;
            return Ok(mynum1{
                a:a,
                b:b
            });
        }
    }
    let mn:Result<mynum1, &str>="1,1".to_string().try_into();
    println!("{:?}",mn.unwrap());
    
    let a:i32=1;
    let b:usize=1;
    if(a==(b as i32)){要强制转义才可以
        //
    }
    
}

```
