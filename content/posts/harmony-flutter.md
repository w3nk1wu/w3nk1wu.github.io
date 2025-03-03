---
title: "Harmony Flutter"
date: 2025-03-03T10:02:57+08:00
draft: true
---


## 开发前准备
1、下载 OpenHarmony 版 flutter，并配置环境变量  
 [下载地址](https://gitcode.com/openharmony-sig/flutter_flutter)

```
# 在用户变量中新建
PUB_HOSTED_URL=https://pub.flutter-io.cn  # 国内镜像
FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

# 在 path 变量中添加
D:/HuaWei/flutter_flutter/bin # flutter_flutter 的下载目录
```
2、其它环境  
需要 JAVA 环境，下载 JDK 并配置 JAVA 环境变量  
需要下载安装 DevEco Studio 开发工具  
需要设置 OpenHarmony 开发环境变量
```
# 在用户变量中新建
TOOL_HOME=D:\Program Files\Huawei\DevEco Studio
DEVECO_SDK_HOME=D:\Program Files\Huawei\DevEco Studio\sdk # /sdk

# 在 path 变量中添加
%TOOL_HOME%\tools\ohpm\bin # ohpm 
%TOOL_HOME%\tools\hvigor\bin # hvigor
%TOOL_HOME%\tools\node\bin # node
```
3、配置完成可使用 `flutter doctor -v` 检查环境变量配置是否正确

4、使用下面命令创建flutter工程  
```
flutter create --platform ohos flutter_ohos_demo
```
5、项目签名  
在运行项目前，需要通过 __DevEco Studio__ 对项目进行签名  
使用 __DevEco Studio__ 打开项目的 `ohos` 模块，通过 __Project Structure__ 的 __Signing Configs__ 配置签名，勾选 __Automatically generate signature__ 登陆账号进行自动签名  

6、编译运行  
进入 `flutter_ohos_demo` 工程目录，运行到手机
```
flutter run --debug
```
或者直接在 DevEco Studio 中运行 ohos 项目

## 插件开发
1、构建插件模块  
支持在原有的插件工程中[添加 ohos 插件模块](https://gitee.com/openharmony-sig/flutter_samples/blob/master/ohos/docs/04_development/%E5%BC%80%E5%8F%91plugin.md)
```
flutter create . --template=plugin --platforms=ohos
```
需要修改插件工程的 `pubspec.yaml` 文件，添加插件平台 ohos 
添加插件平台的插件模块，最后实现


2、flutter.har 的获取  
在 flutter_flutter 的 __\bin\cache\artifacts\engine__ 目录中，将ohos-arm64-release下的`flutter.har`复制出来  
## 插件使用
使用 DevEco Studio 打开 example下的 ohos 工程  
1、修改 oh-package.json5文件，添加依赖
```
  "dependencies": {
    "@ohos/flutter_ohos": "file:./har/flutter.har",
    "openinstall_flutter_plugin": "file:../../ohos"
  },
```
2、在根目录新建文件夹 har，并将上面复制的 flutter.har放入文件夹中  
3、判断当前运行平台是否是harmony
``` dart
import 'package:flutter/foundation.dart';

bool isOhos() {
  return defaultTargetPlatform == TargetPlatform.ohos;
}
```


