---
proc-macro 使用后不能使用macro_rules!定义
---


引入包

```
[lib]
proc-macro = true
[dependencies]
syn = {version = "~1.0.74",features = ["full"]}
quote = "~1.0.9"
proc-macro2 = "~1.0.28"
```

3种类型宏定义和使用

```rust
use proc_macro::{TokenStream};
use quote::quote;
use proc_macro2::{Ident, Span};
use syn::{DataStruct, Data, Fields, parse_macro_input, DeriveInput, Meta, NestedMeta};

//类函数宏
#[proc_macro]
pub fn my_proc_macro(ident: TokenStream) -> TokenStream {
    let new_func_name = format!("test_{}", ident.to_string());
    let concated_ident = Ident::new(&new_func_name, Span::call_site());
    let expanded = quote! {
        fn #concated_ident<T: std::fmt::Debug>(t: T) {
            println!("{:?}", t);
        }
    };
    expanded.into()
}
//使用示例
// untitled::my_proc_macro!(hello);


// syn::parse_macro_input
// 1. 方法返回的可以直接用在 quote!{} 里面,整个输入的数据
// 2. 返回按类型生成的结构体,可以访问里面内容进行获取数据,如结构体名,函数名等

// quote::format_ident
// 1. 可将字符串转为token

//继承宏,确定不会有参数时使用
#[proc_macro_derive(Show)]
pub fn derive_show(item: TokenStream) -> TokenStream { //生成的结果不用 derive_show
    // 解析整个token tree
    let input = parse_macro_input!(item as DeriveInput);
    let struct_name = &input.ident; // 结构体名字
    // 提取结构体里的字段
    let expanded = match input.data {
        Data::Struct(DataStruct{ref fields,..}) => {
            if let Fields::Named(ref fields_name) = fields {
                // 结构体中可能是多个字段
                let get_selfs: Vec<_> = fields_name.named.iter().map(|field| {
                    let field_name = field.ident.as_ref().unwrap(); // 字段名字
                    quote! {
                        &self.#field_name
                    }
                }).collect();

                let implemented_show = quote! {
                // 下面就是Display trait的定义了
                // use std::fmt; // 不要这样import，因为std::fmt是全局的，无法做到卫生性(hygiene)
                // 编译器会报错重复import fmt当你多次使用Show之后
                impl std::fmt::Display for #struct_name {
                    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
                        // #(#get_self),*，这是多重匹配，生成的样子大概是这样：&self.a, &self.b, &self.c, ...
                        // 用法和标准宏有点像，关于多个匹配，可以看这个文档
                        // https://docs.rs/quote/1.0.0/quote/macro.quote.html
                        write!(f, "{} {:?}", stringify!(#struct_name), (#(#get_selfs),*))
                    }
                }
            };
                implemented_show

            } else {
                panic!("sorry, may it's a complicated struct.");
            }
        }
        _ => panic!("sorry, Show is not implemented for union or enum type.")
    };
    expanded.into()
}
// 使用示例
// #[derive(untitled::Show)]
// struct A {}



// 属性宏 有参数时使用
#[proc_macro_attribute]
pub fn run_time(args: TokenStream, item: TokenStream) -> TokenStream {

    // 作用的结构体上
    //let func = syn::parse_macro_input!(item as syn::DeriveInput);

    // 作用在函数体上
    let func = syn::parse_macro_input!(item as syn::ItemFn);
    let func_vis = &func.vis; // pub
    let func_block = &func.block; // 函数主体实现部分{}
    let func_decl = &func.sig; // 函数申明
    let func_name = &func_decl.ident; // 函数名
    let func_generics = &func_decl.generics; // 函数泛型
    let func_inputs = &func_decl.inputs; // 函数输入参数
    let func_output = &func_decl.output; // 函数返回
    // 提取参数，参数可能是多个
    let params: Vec<_> = func_inputs.iter().map(|i| {
        match i {
            // 提取形参的pattern
            // https://docs.rs/syn/1.0.1/syn/struct.PatType.html
            syn::FnArg::Typed(ref val) => &val.pat, // pat没有办法移出val，只能借用，或者val.pat.clone()
            _ => unreachable!("it's not gonna happen."),
        }
    }).collect();

    // attr 为 untitled::run_time(args) 中的args
    let args = syn::parse_macro_input!(args as syn::AttributeArgs);
    // 提取attr的ident，此处例子只有一个attribute
    let tmp= args.get(0).unwrap();
    let attr_ident = match tmp {
        //按类型匹配 args,可以是路径,键值对等
        NestedMeta::Meta(Meta::NameValue(ref attr_ident)) => attr_ident.clone(),
        _ => unreachable!("it not gonna happen."),
    };
    //attr_ident.path 参数名 例如 #[untitled::run_time(add=1)] 中的add
    //attr_ident.lit 参数值 例如 #[untitled::run_time(add=1)] 中的 1
    let add_num= match attr_ident.lit {
        syn::Lit::Int(val)=>val,
        _=>unreachable!("it not num"),
    };
    // 创建新的ident, 例子里这个ident的名字是time_measure
    // let attr = Ident::new(&attr.to_string(), Span::call_site());
    let expanded = quote! { // 重新构建函数执行
        #func_vis fn #func_name #func_generics(#func_inputs) #func_output {
            fn rebuild_func #func_generics(#func_inputs) #func_output #func_block
            // 要修饰函数的参数，有可能是多个参数，所以这样匹配 #(#params,) * //为vec时候合并输出
            let mut secs=rebuild_func(#(#params),*);
            use std::ops::Add;
            let addse =#add_num;
            secs=secs.add(std::time::Duration::from_secs(addse));
            secs
        }
    };
    expanded.into()
}

//使用示例
// #[untitled::run_time(add=1)]
// fn get_duration(t: u64) ->std::time::Duration{
//     let secs = std::time::Duration::from_secs(t);
//     secs
// }
```
