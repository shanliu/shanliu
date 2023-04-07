#react-native

安装命令行工具

> npm i react-native-cli -g

初始化项目

> react-native init

安装debug版本[android]

> react-native run-android


手动启动调试服务器[一般运行完react-native run-android会启动]

> react-native start

签名android程序 `其中　release　替换你的秘钥名`

> keytool -genkey -v -keystore shan-release.keystore -alias shan-release -keyalg RSA -keysize 2048 -validity 10000


生成秘钥放入目录 ./android/app 中 并修改该目录下的文件:build.gradle

```
signingConfigs {
    release {
        storeFile file("shan-release.keystore")
        storePassword "000000"
        keyAlias "shan-keystore"
        keyPassword "000000"
    }
}
 buildTypes {
    release {
        ．．．
    signingConfig signingConfigs.release
    }
}
```

在./android 目录下执行[如果不加秘钥也可以生成未签名的apk,可能无法安装]

> ./gradlew assembleRelease


重新打包可尝试清理，命令如下:

>./gradlew clean


调试 一般8081启动时候会自动映射而React Inspector使用的8097未开启映射 开启端口映射命令,开启8097后方便调试样式

> adb reverse tcp:8081 tcp:8081

> adb reverse tcp:8097 tcp:8097

模块冲突解决
error: bundling failed: ambiguous resolution: module

> npm start -- --reset-cache
