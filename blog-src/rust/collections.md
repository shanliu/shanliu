---
集合 [collections]
---


```rust

use std::collections::HashMap;
fn main() {
    let bb=[1,2,3];//数组
    //let t=bb[4];//抛出异常,不建议直接访问
    let mut bbv =Vec::from(bb);
    //查找
    let t=bbv.binary_search(&1);
    if t.is_ok() {
        println!("{:?}",t.unwrap());
    }
    //增加
    bbv.insert(0,5);
    bbv.push(11);
    bbv.extend(vec![1,2,3]);//批量加
    //删除
    bbv.pop();
    bbv.remove(bbv.len()-1);
    //获取
    println!("{:?}",bbv.get(5));//返回Option
    bbv[1]=111;//注意是否超范围
    println!("{:?}",bbv.get(5));//转为vec后可抗访问
    //转为迭代器
    let t=bbv.into_iter();
    //从迭代器还原
    let tt=t.collect::<Vec<i32>>();
    println!("{:?}",tt);//转为vec后可抗访问
    
    let mut u=vec![1,2,4];
    //let t=u[10];//下标引用超范围触发panic，跟切片 数组一样，用get处理
    println!("{:?}",u);
    let mut u1=&u[0..2];//直接获取切片
    println!("{:?}",u1);
    let ss=u.drain(1..3);//获取指定范围迭代器并清理调该数据
    println!("{:?}",ss);
    drop(ss);//结束可变引用
    println!("{:?}",u);//剩余未被drain数据
    
     let m=String::from_utf8(Vec::from(b"dddddddd" as &[u8])).expect("dddddd");
    let o=&m[0..2];//字符串内部由vec[u8]组成
    
    let result = panic::catch_unwind(|| {
        let m=10;
        let s=&u[0..m];//超范围访问捕捉
    });
    println!("{:?}",result);
    
    //合并迭代器
    let chaint=vec![1,2].into_iter().chain(vec![2,3,4].into_iter());
    for out in chaint {
        println!("{}",out);
    }
    //待索引迭代
    let it=vec![1,2,3].into_iter().enumerate();
    for (a,b) in it {
        println!("i:{} data:{}",a,b);
    }
    //转为map迭代器
    let map=vec![1,2,3].into_iter().map(|e|{
       println!("map:{}",e);
    });
    map.collect::<Vec<_>>();//MAP迭代器进行迭代

    //附带参数迭代器
    let scan=vec![1,2,3].into_iter().scan(10,|initdata,param|{
        //initdata 提供的附带数据 10
        //param 迭代数据
        if param==1 {
            return Some(param);
        }else{
            return None;//返回0停止迭代
        }
    });
    let out=scan.collect::<Vec<_>>();//scan迭代器进行迭代
    println!("{:?}",out);

    //两层vec结构转一层
    let out=vec![vec![1, 2, 3, 4], vec![5, 6]].into_iter().flatten();
    println!("{:?}",out.collect::<Vec<_>>());

    //无限循环不退出,用在对结果多次遍历
    let out=vec![1,2,3].into_iter().cycle();
    let mut skip =true;
    for t in out {
        println!("cycle:{}",t);
        if t>=3 { skip=false};
        if !skip { break;}
    }
    //以下两个方法配合用于获取指定范围的迭代器
    //skip 跳过指定迭代数量
    //take 进行迭代数量
    let iter = vec![2,2,3,4,5].into_iter().skip(2).take(1);
    for t in iter {
        println!("take:{}",t); 
    }

    //出现None后,以后都返回none,在自定义迭代器时用
    let iter =vec![1,1].into_iter().fuse();


    //一个为真返回为真
    let any =vec![1,1].into_iter().any(|x|{return x>0});

    //跟scan类似.函数式累加
    let fold=vec![1,2].into_iter().fold("dd".to_string(),|xx,x|{
        return format!("{}{}",xx,x);
    });
    println!("fold:{}",fold);

    //any collect fold 属于迭代消费器

let a1 = [1, 2, 3];
let a2 = [4, 5, 6];

let mut iter = a1.iter().zip(a2.iter());

assert_eq!(iter.next(), Some((&1, &4)));
assert_eq!(iter.next(), Some((&2, &5)));
assert_eq!(iter.next(), Some((&3, &6)));
assert_eq!(iter.next(), None);


    let mut map = HashMap::new();
    map.insert("a", 1);
    map.insert("b", 2);
    let iter = map.iter();//迭代器
    for (key,val) in iter {
        println!("key:{}-val:{}",key,val);
    }
    println!("{:?}",map.keys());//返回KEY
    let iter = map.drain();//清空并创建迭代器 可变借用,迭代前不可访问
    for (key,val) in iter {
        println!("key:{}-val:{}",key,val);
    }
    println!("{:?}",map);

    //字符串迭代器处理
    let bb="tt|tt";
    let cc="tt1|tt1";
    let split=bb.split("|");
    println!("{:?}",split.clone().collect::<Vec<&str>>());
    let chain=split.chain(cc.split("|"));
    println!("{:?}",chain.collect::<Vec<&str>>());

}
```

自定义迭代器

```rust
fn main() {
    //自定义迭代器实现
    struct aa<i32>{
        data:Vec<i32>,
        iter:usize
    }
    impl Iterator for aa<i32>{
        type Item = i32;
        fn next(&mut self) -> Option<Self::Item> {
            if self.iter<self.data.len() {
                let t: i32 =self.data[self.iter];
                self.iter+=1;
                return Some(t);
            }else{
                return None;
            }
        }
        //
        fn size_hint(&self) -> (usize, Option<usize>) {
             (self.data.len(), None)
             //已明确知道的元素数量，可能的最大元素数量（无法确认none）
             //未迭代的迭代器 已知道元素为0
             //但可能最大元素为总数
         }
    }
    let b=aa{
        data:vec![1,1,2],
        iter:0
    };
    for cc in b {
        println!("{}",cc);
    }
}
```

迭代时元素增加代替

```rust
fn main() {
    let a=vec![1,2];//进行迭代时没所有权,不能往vec增加元素
    let mut tmp:Vec<i32>=vec![];//所以使用另一个vec来存储迭代时新增要迭代数据
    // 等于再次迭代这个新增的迭代器里数据来模拟往a里新增效果
    loop{
        let lop;
        if tmp.len()>0 {
            lop=tmp.clone().into_iter();
            tmp.clear();
        }else{
            lop=a.clone().into_iter();
        }
        for core in lop {
            if core<=1 {//符合某些规则时新增迭代数据
                tmp.push(10);//继续往下处理迭代的新增数据
            }
        }
        if tmp.len()==0 {break}
    }
}
```
