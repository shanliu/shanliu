---
宏示例 
---

> macro_rules


```rust
#![feature(trace_macros)]
//1. 确定宏生成结果,确认循环点 [输出结果]
// let out={
//     let len=<[()]>::len([()]);//内部()循环N次
//     let mut a =::std::collections::HashMap::with_capacity(len);
//     a.insert(1,2);//循环N次
//     a
// };
//2. 确定调用示例 [输入样式]
//let out=myhash!{1=>2,3=>3};
//3. 合并两者
macro_rules! myhash {
    //通过特别前缀的输入规则来拆分多个宏
    (tounit $push_key:tt)=>{()};//输入任何数据都输出(),生成跟输入无关数据
    (count $($push_key:tt),*)=>(//不能跨宏定义变量在使用
         <[()]>::len(&[$( myhash!(tounit $push_key) ,)*]);
    );
    ($($push_key:expr=>$push_val:expr),* $(,)?) //根据输入示例编写输入规则
    =>{ //根据输出结果写输出
        {//这个非必须.根据示例生成的
          let len= myhash!(count $($push_key),*);//如果len在 myhash!(count )里定义 下面使用将报错
          let mut a =::std::collections::HashMap::with_capacity(len);
          $(a.insert($push_key,$push_val);)* //这个根据结果生成
          a
        }
    };
}

// $($push_key:expr),*
// $(匹配关键字:匹配类型)连接字符 出现次数
// 上面只匹配 a,b 而不匹配 a,b, 因为多了 , 结尾

macro_rules! lopreg {
    ($($push_key:expr),* $(,)?)
    // 通过$push_key在下面使用 $(,) 无法在内容中使用,但可用于匹配输入
    //匹配只能用于全匹配
    =>{
        $($push_key);+ ;//用;连接,最后一个没有;
        $($push_key;)+ //每个元素都有 ;
        //最后一个+ 用于输出次数,但循环有输入解决等于制约输入循环次数
    };
}


fn main() {
    lopreg!(1,2,3);
    //4. 测试宏
    //trace_macros!(true); //开启调试,影响当前及以下mod,上层不影响
    // cargo rustc -- -Z unstable-options --pretty=expanded 展开宏后代码
    // 存在lib.rs 跟 main.rs 时指定编译目标
    // cargo rustc --lib  -- -Z unstable-options --pretty=expanded 
    // cargo rustc --bin untitled -- -Z unstable-options --pretty=expanded
    //或使用 cargo expand 但宏有问题时输出异常,并无提示
    // myhash!{1=>2,3=>3,2=>1};
    // a::a();
}
mod a{
    pub fn a(){
        myhash!{1=>2};
    }
}
```
