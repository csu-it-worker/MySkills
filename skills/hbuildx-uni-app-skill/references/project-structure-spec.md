# uni-app x 项目结构规范总结

## 一、项目创建

1. **环境要求**：需使用最新版 HBuilder X
2. **创建方式**：菜单 `文件 > 新建 > 项目`，勾选 `uni-app x` 选项（注意区分普通 uni-app 项目）
3. **项目标识**：`manifest.json` 中会包含 `"uni-app-x" : {}` 节点，这是识别项目类型的标记
4. **支持版本**：仅支持 Vue 3，不支持 Vue 2

## 二、项目结构

uni-app x 项目结构与传统 uni-app 基本一致，主要区别是不支持 `nativeplugins` 目录（不支持 App 原生语言插件，仅支持 uts 插件）。

```
┌─uniCloud              云空间目录（根据服务商不同分为 uniCloud-alipay/aliyun/tcb）
│─components            符合 Vue 组件规范的可复用组件目录
├─utssdk                存放 uts 文件（已废弃）
├─pages                 业务页面文件目录
│  ├─index
│  │  └─index.uvue      index 页面
│  └─list
│     └─list.uvue       list 页面
├─static                本地静态资源目录（图片、字体、音视频等，必须存放于此）
├─uni_modules           存放 uni_module 插件
├─platforms             各平台专用页面目录
├─nativeResources       App 端原生资源目录
│  ├─android            Android 原生资源目录
│  └─ios                iOS 原生资源目录
├─harmonyConfig         HarmonyOS 端原生资源配置目录
├─hybrid                App 端存放 web-view 组件使用的本地 HTML 文件目录
├─wxcomponents          微信小程序平台 wxml 组件专用目录
├─unpackage             存放编译结果、自定义基座等非工程代码，建议 git 忽略
├─main.uts              Vue 初始化入口文件
├─App.uvue              应用配置，配置全局样式、监听应用生命周期
├─pages.json            配置页面路由、导航条、选项卡等页面信息
├─manifest.json         配置应用名称、appid、logo、版本等打包信息
├─AndroidManifest.xml   Android 原生应用清单文件
├─Info.plist            iOS 原生应用配置文件
└─uni.scss              内置常用样式变量
```

> 开发提示：可在项目根目录添加 `.cursor` 目录帮助 AI 工具更好生成 uni-app x 代码

## 三、运行机制

### 1. 应用实例
每个 uni-app x 应用启动后会创建一个 UniApp 实例，通过全局 API `getApp()` 获取，实例上挂载应用级的方法和属性。

### 2. 运行方式
- **运行入口**：顶部菜单、工具栏运行按钮、快捷键 `Ctrl + r` 三种方式
- **运行环境**：不同目标平台需要对应环境（浏览器、小程序模拟器、手机真机/模拟器等）

### 3. 运行到 App
- **Android/iOS**：提供标准真机运行基座（包名 `io.dcloud.uniappx`），支持修改代码后热刷新
  - 标准基座：使用 DCloud 提供的默认配置
  - 自定义基座：当需要自定义包名、证书、三方 SDK 配置时使用，打包后存放于 `unpackage` 目录
- **鸿蒙**：无基座概念，直接调用本地 DevEco 出包，不支持热刷新
- **日志**：默认仅显示开发者代码和框架日志，可手动开启原生日志

### 4. 运行到 Web
- 基于 Vite 实现按需编译，首页启动快，后续持续编译其他页面
- 所有页面错误都会在控制台输出，即使未打开该页面

## 四、发行说明

### 1. 发行方式
- 入口：顶部菜单 `发行` 或右键项目，选择目标平台
- 支持本地发行（需要本地配置发布环境），Android/iOS 额外支持云打包（无需配置原生环境，直接输出 apk/ipa）
- 支持通过 HBuilderX CLI 实现持续集成和自动发布

### 2. 特殊发行场景
- **作为原生应用的一部分**：
  - App 平台：可发布为 Kotlin/Swift 源码，集成到现有原生项目（参考 uni-app x 原生 SDK 文档）
  - 小程序平台：可发布为分包，集成到其他小程序中
- **渐进式集成**：适合已有非 uni-app x 应用，使用 uni-app x 开发部分页面逐步集成

## 五、关键区别总结

| 特性 | uni-app x | 传统 uni-app |
|------|-----------|--------------|
| Vue 版本 | 仅支持 Vue 3 | 支持 Vue 2/Vue 3 |
| 原生插件 | 仅支持 uts 插件 | 支持原生插件 + uts 插件 |
| App 基座 | 独立基座（包名 io.dcloud.uniappx） | HBuilder 基座 |
| 鸿蒙运行 | 直接调用 Deveco 出包 | - |
| Web 编译 | 基于 Vite 按需编译 | - |