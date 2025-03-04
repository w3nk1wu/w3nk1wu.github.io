---
title: "Harmony React Native"
date: 2025-03-03T15:50:51+08:00
draft: false
tags: ["Harmony", "React Native"]
---

## 开发环境
1、鸿蒙开发工具 DevEco Studio  
前往 [华为开发者联盟](https://developer.huawei.com/consumer/cn/download/) 下载最新版的 DevEco Studio  
2、添加 node 环境变量  
在系统或者用户的 PATH 变量中，添加 Node.js 安装位置的路径（默认路径为 $DevEco Studio安装目录/tools/node 下）  
3、配置 hdc 环境变量
鸿蒙 React Native 工程使用 hdc 进行真机调试。  
hdc 工具存放于 SDK 的 toolchains 目录下，默认路径为 {DevEco Studio安装路径}/sdk/{SDK版本}/openharmony/toolchains  
添加 HDC 端口变量名为：HDC_SERVER_PORT  
4、配置 CAPI 版本环境变量  
在系统变量中点击新建，添加变量名为：RNOH_C_API_ARCH，变量值为 1  

## React Native 工程配置
1、创建Rect Native 项目，目前 React Native for OpenHarmony 仅支持 0.72.5 版本
```
npx react-native@0.72.5 init AwesomeProject --version 0.72.5
```
2、在 AwesomeProject 目录下运行安装依赖包命令
```
npm i @react-native-oh/react-native-harmony
```
3、打开 AwesomeProject 目录下的 package.json，在 scripts 下新增 OpenHarmony 的依赖
``` diff
 "scripts": {
   "android": "react-native run-android",
   "ios": "react-native run-ios",
   "lint": "eslint .",
   "start": "react-native start",
   "test": "jest",
+    "dev": "react-native bundle-harmony --dev"
 },
```
4、打开 AwsomeProject\metro.config.js 文件，并添加 HarmonyOS 的适配代码。
```
const {mergeConfig, getDefaultConfig} = require('@react-native/metro-config');
const {createHarmonyMetroConfig} = require('@react-native-oh/react-native-harmony/metro.config');

/**
* @type {import("metro-config").ConfigT}
*/
const config = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: true,
      },
    }),
  },
};

module.exports = mergeConfig(getDefaultConfig(__dirname), createHarmonyMetroConfig({
  reactNativeHarmonyPackageName: '@react-native-oh/react-native-harmony',
}), config);
```
5、在 AwesomeProject 目录下运行生成 bundle 文件的命令。
```
npm run dev
```

## 鸿蒙工程配置
1、使用 DevEco Studio 创建空的鸿蒙工程 Mypplication  
创建完成后，先进行签名配置  
2、在 entry 目录下执行以下命令：
```
ohpm i @rnoh/react-native-openharmony
```
3、根据[官方文档](https://gitee.com/openharmony-sig/ohos_react_native/blob/0.72.5-ohos-5.0-release/docs/zh-cn/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.md#%E5%9C%A8%E5%8E%9F%E7%94%9F%E5%B7%A5%E7%A8%8B%E4%B8%AD%E9%9B%86%E6%88%90rnoh)添加 CPP 代码和 Arkts 代码  
注意 __MyApplicaton\entry\src\main\ets\pages\Index.ets__ 文件中 `RNApp` 的参数 `appKey` 需要与 RN 工程中 `AppRegistry.registerComponent` 注册的 `appName` 保持一致  
4、加载bundle包  
本地加载bundle：将使用 `npm run dev` 生成的 bundle 文件（位于 __AwesomeProject/harmony/entry/src/main/resources/rawfile__ 文件夹下）拷贝到鸿蒙工程的 __entry/src/main/resources/rawfile__ 路径下  
使用 Metro 服务加载：使用 `MetroJSBundleProvider.fromServerIp("192.168.0.xxx", 8081)` 可以在手机wifi和电脑处于同一网络下，不需要连接数据线和转发端口，即可访问到 Metro 服务  

## TruboModule 模块开发
1、声明 JavaScript 接口  
在 RN 工程根目录同一目录创建模块文件夹（假如 XYZ），并在 `XYZ/src` 目录中创建 ts 文件  
2、配置 Codegen  
在模块根目录创建 pacakge,json 文件，并配置 codegen
```
  {
    ...
    "harmony": {
      "codegenConfig": {
        "specPaths": [
          "./src"
        ]
      }
    }
  }
```
3、在根目录使用 `npm pack` 打包模块，将会生成 XYZ-a.b.c.tgz 文件  
4、打开 RN 工程下的 package.json，添加：  
``` diff
{
  ...
  "scripts": {
    ...
+    "codegen": "react-native codegen-harmony --rnoh-module-path ./harmony/react_native_openharmony"
  },
  ...
}
```
> --rnoh-module-path: 指定 rnoh 模块的相对路径，用于存储生成的 ts 文件；如果使用 har 包引入 rnoh 模块，则需要指向：./harmony/entry/oh_modules/@rnoh/react-native-openharmony"  

5、使用命令将 XYZ 模块添加到 RN 项目中  
```
npm i file:../XYZ/XYZ-a.b.c.tgz
```
6、执行 codegen 生成串联文件
```
npm run codegen
```
## 鸿蒙原生代码实现
1、使用 DevEco Studio 打开 MyApplication 工程，并创建 **Static Library** 模板的 Module，并删除无用文件  
2、修改模块下的 oh-package.json5文件，添加RNOH依赖  
```
{
  "name": "xyz",
  "version": "1.0.0",
  "description": "Please describe the basic information.",
  "main": "Index.ets",
  "author": "",
  "license": "Apache-2.0",
  "dependencies": {
    "@rnoh/react-native-openharmony": "^0.72.38"
  }
}
```
3、代码实现并导出  
模块实现文件需要继承 `TurboModule` 并实现 codegen 生成的 `TM.Xyz.Spec` 的接口文件  
先创建 ts.ts 文件导出，再创建 index.ets 导出



------

## 参考文档
[开发环境搭建](https://gitee.com/openharmony-sig/ohos_react_native/blob/0.72.5-ohos-5.0-release/docs/zh-cn/%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA.md)  
[TurboModule开发文档](https://gitee.com/react-native-oh-library/docs/blob/master/zh-cn/turbomodule.md)  
[Metro热加载](https://gitee.com/openharmony-sig/ohos_react_native/blob/0.72.5-ohos-5.0-release/docs/zh-cn/%E8%B0%83%E8%AF%95%E8%B0%83%E6%B5%8B.md#metro%E7%83%AD%E5%8A%A0%E8%BD%BD)  





































