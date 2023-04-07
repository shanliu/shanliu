# creat

> 添加编译目标支持

```
rustup target add wasm32-wasi
```

> 编译指定平台目标

```
cargo run --target wasm32-wasi
```

> 扩展宏

```
cargo +nightly install cargo-expand
```

> `futures` 默认包含运行时和channel `futures-executor` 不需要可用 `futures-util` \[例如用运行时:async-std,tokio,运行时一般也内置channel]  `futures-channel` 被导出为 `channel`

> `async-std` 默认不包含 #\[async\_std::main],需要指定 features `attributes`

> `tokio` 默认不包含 #\[tokio::main],需要指定 features `macros`

cargo.toml&#x20;

```
#多个crate时使用,一般会将项目拆分成多个crate
[workspace]
members = [
    ".",
    "m1"
]

[package]
name = "untitled2"
version = "0.1.0"
edition = "2018"
# 默认运行的入口
default-run="cool"
[dependencies]
m1 = { version = "0.1.0", path = "./m1"}


# 多个可执行文件配置
[[bin]]
name = "cool"
path = "src/main.rs"

[[bin]]
name = "frobnicator"
path = "src/main1.rs"

```
