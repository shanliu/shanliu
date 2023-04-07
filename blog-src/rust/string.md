---
 字符串
---


字符串操作

```rust
#![allow(unused)]
fn main() {

    //其他类型格式化为字符串
    println!("{}",format!("test{},hi",11));

    let mut str1 ="的te都st".to_string();
    //str1==str2
    //UTF8获取和查找
    println!("{}",str1.get(0..3).unwrap());//取内部某给部分,完整字符边界,否则出错
    //包含字符
    println!("{:?}",str1.contains("t"));//是否包含某字符
    println!("{:?}",str1.contains(char::is_lowercase));//通过回调函数判断,是否存在小写字符
    println!("{:?}",str1.starts_with("的"));//是否某字符串开头
    println!("{:?}",str1.ends_with("st"));//是否某字符串结尾
    //查找字符
    println!("{:?}",str1.find("t").unwrap_or(0));//正向查找某字符,完整字符边界,否则出错
    println!("{:?}",str1.rfind("t").unwrap_or(0));//反向查找某字符,完整字符边界,否则出错
    //拆分
    println!("{:?}",str1.split_at(3));//完整字符边界,否则出错
    println!("{:?}",str1.split("s"));//完整字符边界,否则出错
    println!("{:?}",str1.split_terminator("s"));//完整字符边界,否则出错
    println!("{:?}",str1.rsplit("s"));//完整字符边界,否则出错
    //模式匹配
    let mstr="cccbbbccc".to_string();
    let matstr=mstr.matches("c").collect::<Vec<&str>>();//得到匹配的集合
    println!("{:?}",matstr);
    let matstr=mstr.rmatches("c").collect::<Vec<&str>>();//反向得到匹配的集合
    println!("{:?}",matstr);
    let matstr=mstr.match_indices("c").collect::<Vec<_>>();//反向得到匹配的索引和集合
    println!("{:?}",matstr);
    let matstr=mstr.rmatch_indices("c").collect::<Vec<_>>();//反向得到匹配的索引和集合
    println!("{:?}",matstr);

    //UTF8添加
    str1.push('c');//添加字符
    str1.push_str("cc");//添加字符串
    str1.extend(vec!['a','b','c']);//添加迭代器字符
    //自定位置添加
    str1.insert(0,'a');
    str1.insert_str(0,"开头00");
    //连接符添加
    str1+="++";

    //UTF8删除
    let pchar=str1.pop();//删除最后一个字符
    str1.remove(0);//删除并返回指定字符
    let tmp=str1.drain(0..5);//删除并返回自定字符串drain 完整字符边界
    let tmpstr:String=tmp.collect();//转为字符串
    println!("{}",tmpstr);
    str1.truncate(5);//截断到指定长度 完整字符边界
    println!("{}",str1);
    //匹配两边删除 trim 系列函数
    let delstr="sssbbbccc".to_string();
    println!("{}",delstr.trim_matches('c'));

    //UTF8字符替换
    let delstr="sssbbbbccc".to_string();
    println!("{}",delstr.replace("bb","kk"));//全量替换
    println!("{}",delstr.replacen("c","s",1));//限制次数替换

    //字符串常用迭代
    //迭代一般为转为其他形态做中间变量,如转为容器或截取部分迭代内容
    //转为字符迭代
    let chars=str1.chars();
    for mychar in chars {
        println!("{}",mychar);
    }
    //转为字符迭代 (字符长度,字符)
    let chars=str1.char_indices();
    for mychar in chars {
        println!("{:?}",mychar);
    }
    //转为U8[整数]迭代
    let chars=str1.bytes();
    for mychar in chars {
        println!("{}",mychar);
    }

    //字符串安全操作转换: 类型<-collect->迭代器<-collect->Vec<char|u8>
    //替换字符串
    let mut b ="sss发的bbb".chars().collect::<Vec<char>>();
    //修改容器里的值
    b[4]='非';
    //容器还原为迭代器,在把迭代器转为String
    let t=b.into_iter().collect::<String>();
    println!("{}",t);

    //范围获取字符串
    let mut s = String::from("α is alpha, 的 is beta");
    let mut s1 =s.chars().collect::<Vec<char>>();
    let s2=s1.drain(0..2);
    let s2=s2.collect::<String>();
    let s1=s1.into_iter().collect::<String>();
    println!("s1:{:?}\ns2:{:?}",s1,s2);

}
```
