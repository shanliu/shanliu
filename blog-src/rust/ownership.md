---
所有权 ownership
---


结构所有权

```rust
#[derive(Debug)]
struct bbb{
}
//struct enum 默认非copy,需要手动实现
#[derive(Debug,Copy, Clone)]
struct cpbbb{
}
#[derive(Debug)]
struct aaa{
    var1:bbb,
    var2:i32
}
#[derive(Debug,Copy, Clone)]
struct cpaaa{
    //var1:bbb,//如果外部可copy,所有内部也必须可copy,bbb不可copy所以错误
    var2:cpbbb
}
fn tt(c:aaa){
    println!("{:?}",c);
}
fn main() {
    //----------------------
    let a=cpaaa{
        var2: cpbbb{}
    };

    //----------------------
    //数组,元组,Option如果内值可复制,则实现复制
    let a=[bbb{}];
    let b=a;//bbb 非复制,所以a所有权转移到b
    //println!("{:?}",a);//a 未定义
    let ca=[cpbbb{}];
    let cb=ca;
    println!("{:?}",ca);//cb复制ca 不转移

    let a=(bbb{});
    let b=a;//bbb 非复制,所以a所有权转移到b
    //println!("{:?}",a);//a 未定义
    let ca=(cpbbb{});
    let cb=ca;
    println!("{:?}",ca);//cb复制ca 不转移

    let a=Some(bbb{});
    let b=a;//bbb 非复制,所以a所有权转移到b
    //println!("{:?}",a);//a 未定义
    let ca=Some(cpbbb{});
    let cb=ca;
    println!("{:?}",ca);//cb复制ca 不转移

    //----------------------
    let mut a =aaa{
        var1:bbb{},
        var2:1
    };
    let b=a.var2;
    let c=a.var1;//结构里的一个所有权被转移会导致整个结构不可用
    a.var1=c;//转移回去会正常
    tt(a);

}
```

变量范围所有权

```rust
#[derive(Debug)]
struct aaa{
    a:i32
}
#[derive(Debug,Copy, Clone)]
struct bbb{
    b:i32
}

fn main() {
    //----------------
    let mut a =aaa{a:1};
    let mut b=bbb{b:1};
    {
        let c=a;//a已转移到c
        let d=b;//可复制,不转移所有权
        //c d 释放
    }
    //println!("{:?}",a);//a 已被转移,未定义
    println!("{:?}",b);


    //--------------------
    let mut a =aaa{a:1};
    {
        let c=a;//a已转移到c
        a=aaa{a:1};//a重新被赋值
    }
    println!("{:?}",a);//a已重新赋值,存在

    //-------------
    let mut a =aaa{a:1};
    {
        let c=&mut a;//未被转移,可变引用
        c.a=2;
    }
    println!("{:?}",a);//a已重新赋值,存在 aaa{a:2}

    //-------------
    //转移到空变量
    let t =aaa{a:1};
    t;//注意 t被转移 之后不在可用
    //let c=t;//t 不在可用
    let t =aaa{a:1};
    {
        t;//t被转移 之后不在可用
    }
    //println!("{:?}",t);//t 已在{}被转移

    //-------------
    //闭包
    let a=aaa{a:1};
    let b=bbb{b:1};
    let fc=||{
        //对于复制语义类型，以不可变引用（&T）来进行捕获
        //生成闭包结构类似为 b=b
        let t=b;
        //执行移动语义，转移所有权来进行捕获
        ////生成闭包结构类似为 a=a
        let t1=a;
        //let t1=&a;//可以用引用方式来实现不转移所有权
    };
    fc();
    println!("{:?}",b);
    //println!("{:?}",a);//a已被转移到闭包,a不在可用

    //闭包
    let mut a =aaa{a:1};
    let mut b=bbb{b:1};
    let mut fc=||{
        //可变绑定,在闭包中进行修改的操作，以可变引用捕获
        //生成闭包结构类似为 &mut b=b
        b.b=2;
        //生成闭包结构类似为 &mut a=a
        a.a=2;
    };
    fc();
    println!("{:?}",b);//所有权未被转移
    println!("{:?}",a);//所有权未被转移

    //闭包
    let b =bbb{ b: 0};
    let fc=move ||{
        let t=b;//复制语义不转移所有权
    };
    fc();
    fc();
    println!("{:?}",b);//所有权未被转移

    //闭包
    let a =aaa{ a: 0};
    let fc=move ||{//FnOnce
        let t=a;//非复制语义转移所有权
    };
    fc();
    //fc();//self被消耗
    //println!("{:?}",a);//所有权被转移

    //闭包
    //Fn : FnMut : FnOnce
    let b =bbb{ b: 0};
    let fc=||{//Fn 可多次调用,不改变环境
        let t=b;
    };
    fc();
    fc();
    let mut a =aaa{ a: 0};
    let mut fc=||{//FnMut 可多次调用,改变环境
        a.a=1;
    };
    fc();
    fc();
    let a =aaa{ a: 0};
    let fc=||{//FnOnce 只调用一次,消耗自身
        let t=a;
    };
    fc();
    //fc();//不可二次调用

}
```

