---
RUST与c相互调用
---

Cargo.toml

```
[build-dependencies]
bindgen = "~0.58.1" #绑定C函数到rust
cc = "~1.0.61"#编译C程序成库,编译器功能
#cbindgen = "~0.19" #根据rust库创建C的头文件
[lib] #非必须
crate-type = ["cdylib","lib"] # 如果定义了 crate-type 一定要有lib main.rs才能使用本身 lib.rs中的定义
```

> build.rs

```rust
fn main(){
	//编译环境变量
	//println!("cargo:rustc-env=VAR=VALUE");
	
	//C 编译flags 支持 -l 或 -L 等于使用 rustc-link-lib 或 rustc-link-search
	//println!("cargo:rustc-flags = \"-L /some/path\"");
	
	//RUST 条件编译,等于编译参数 --cfg ,用在条件编译 #[cfg(key)]
	//println!("cargo:rustc-cfg = ['key=\"value\"']");
	//println!("cargo:rustc-cfg = ['key']");
	
	//指定文件变更编译
	println!("cargo:rerun-if-changed=c_src/mytest.c");
	println!("cargo:rerun-if-changed=c_src/mytest.h");
    //使用库
    //库目录 在 /etc/ld.so.conf.d/ 增加SO的加载路径 或者用 `LD_LIBRARY_PATH=./c_src cargo run` 或
    // println!("cargo:rustc-link-search=native={}", std::path::Path::new(
    //     &std::env::var("CARGO_MANIFEST_DIR").unwrap().to_string()
    // ).join("c_src").to_str().unwrap());//添加库目录
    // println!("cargo:rustc-link-lib=dylib=add");//动态库 搜索库目录中的libadd.so 或libadd.dll
    // println!("cargo:rustc-link-lib=static=add");//静态库 搜索库目录中的libadd.a 或libadd.lib
    //从C源码编译库
    cc::Build::new()
        .warnings(true)
        .flag("-Wall")
        .flag("-c")
        .file("c_src/mytest.c")
        .out_dir("my_dir")//自定义共享库生成路径
        .compile("myout");//编译出来的库名,会自动增加到当前编译环境
    //C头文件转为 RUST文件
    let bindings = bindgen::Builder::default()
        .header("c_src/mytest.h")
        .generate()
        .expect("Unable to generate bindings");
    let out_path = std::path::PathBuf::from("src");
    let generated_path = out_path.join("bindcres.rs");
    bindings
        .write_to_file(&generated_path)
        .expect("Unable to write output file");
}
```

> c\_src/mytest.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "mytest.h"
int mytest(int a,char *str){
    char * t="ddd";
    printf("%d,%s,%s",a,str,t);
    return a+1;
}
char *a=NULL;
char* getstr(){
    a=strdup("中文..");
    return a;
}
void freestr(){
    if(!a)free(a);
}
```

> c\_src/mytest.h

```c
#ifndef MYTESTLIB_FFF_H
#define MYTESTLIB_FFF_H
int mytest(int a,char *str);
char* getstr();
void freestr();
#endif //MYTESTLIB_FFF_H
```

> src/lib.rs

```rust
include!("bindcres.rs");//存在main.rs 跟 lib.rs 时,只能在lib.rs 中引用
use std::ffi::CStr;
use std::os::raw::c_char;
#[no_mangle]
pub fn out_str() ->String{
    let c_str: &CStr = unsafe {
        let c_buf: *const c_char =getstr();
        CStr::from_ptr(c_buf)//从C字符串转为RUST的CSTR
    };
    let str_slice:String = c_str.to_str().unwrap().to_owned();//CSTR转为&str在转为String
    println!("form c str:{}",str_slice);
    unsafe {freestr();};//主动释放
    str_slice
}
```

> src/main.rs

```rust
use std::ffi::{CString};
use untitled1::out_str;
use untitled1::mytest;
fn main(){
    let b=out_str();
    println!("rustout:{}",b);
    let str="bb".to_string();
    let c_to_print=CString::new(str).unwrap();//RUST字符串转为 C字符串
    let a;
    unsafe {
        let b=c_to_print.as_ptr() as *mut i8;
        a=mytest(1,b);//一定要在这里进行c_to_print.as_ptr()调用 外部调用出错
    }
    println!("rustout:{}",a);
}
```
