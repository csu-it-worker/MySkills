# uni-app x 文件系统规范总结

## 概述

uni-app x 中，文件主要分为两大类：**代码包文件**和**本地磁盘文件**。开发者可以通过 `uni.getFileSystemManager()` 获取文件系统管理器进行文件操作。

> 注意：`DCloud-`、`DCloud_`、`uni-`、`uni_` 开头的目录和文件是保留目录，开发者应避免使用这些前缀。

---

## 一、代码包目录结构

代码包文件是指 uni-app x 项目中添加的静态资源文件，由编译器打包到发行包中，**只读不可修改**。应用安装后包含4个顶层目录：

| 目录 | 说明 | 访问说明 |
|------|------|----------|
| `assets` | 编译生成的资源目录 | 文件名包含随机数，不推荐通过 FileSystemManager 访问 |
| `hybrid` | 本地HTML文件目录 | 用于 web-view 组件 |
| `static` | 静态资源目录 | 最常被访问，直接使用 `/static/xxx` 路径访问 |
| `uni_modules` | 插件模块目录 | 插件静态资源通过 `/uni_modules/xxx/static/xxx` 路径访问 |

### 访问规范

- 直接使用相对根路径访问，例如：`/static/uni.png`、`/uni_modules/xxx/static/xxx.png`
- 代码包文件只读，如需修改需要先复制到沙盒目录再操作

**示例：将代码包中 static 目录的文件复制到用户数据目录**

```uts
let fileManager = uni.getFileSystemManager()

fileManager.copyFile({
  srcPath: "/static/list-mock/mock.json",
  destPath: `${uni.env.USER_DATA_PATH}/mock.json`,
  success: function (res : FileManagerSuccessResult) {
    console.log('success', res)
  },
  fail: function (res : UniError) {
    console.log('fail', res)
  }
} as CopyFileOptions)
```

### 真机运行特殊说明

真机运行时，为了实现动态更新，代码包文件会被同步到应用沙盒目录：
- Android: `/sdcard/Android/data/%应用包名%/apps/%应用AppID%/www/`
- iOS: `/%应用沙盒目录%/Documents/uni-app-x/apps/%应用AppID%/www/`

> 注意：打包发布后代码包不在沙盒中，请勿依赖真机运行时的目录结构。

---

## 二、本地磁盘目录分类及用途

本地磁盘文件是应用运行时可访问的存储文件，分为**沙盒内**和**沙盒外**。沙盒是应用独立存储区域，不同应用之间隔离。

### 目录结构总览

```
本地磁盘文件
├── 外置应用沙盒目录 (uni.env.SANDBOX_PATH)
│   ├── 缓存目录 (uni.env.CACHE_PATH)
│   └── 用户数据目录 (uni.env.USER_DATA_PATH)
├── 内置应用沙盒目录 (uni.env.ANDROID_INTERNAL_SANDBOX_PATH) [仅Android]
└── 沙盒外目录 [暂不支持API访问]
```

---

### 1. 外置应用沙盒目录 (`uni.env.SANDBOX_PATH`)

- **说明**：App端专有沙盒根目录，可在系统文件管理器中看到
- **平台差异**：
  - Android: `/Android/data/%应用包名%/`
  - iOS: 应用沙盒虚拟目录，包含 Documents、Library、tmp
- **建议**：不推荐直接使用根目录，按需使用下方的缓存目录和用户数据目录

#### 缓存目录 (`uni.env.CACHE_PATH`)

- **用途**：存放应用运行产生的缓存文件
- **特性**：存储空间不足时会被系统自动清理，**不要存放关键业务数据**
- **存储位置**：
  - Android: `/Android/data/%应用包名%/cache/`
  - iOS: `Library/Caches`
- **uni官方预定义目录规范**（开发者避免使用 `uni-` 开头目录）：

| 目录 | 用途 |
|------|------|
| `uni-download` | `uni.downloadFile` 默认下载路径 |
| `uni-media` | 拍摄/选择的图片视频、压缩后的文件、网络图片缓存 |
| `uni-snapshot` | DOM 截图 API 存储路径 |
| `uni-audio` | 网络音频缓存 |
| `uni-recorder` | 录音文件存储 |
| `uni-store` | `saveFile/saveFileSync` 默认保存路径 |
| `uni-crash` | 崩溃日志存储 |

> 提示：cache目录中的文件如果需要长期保存，建议移动到用户数据目录。

#### 用户数据目录 (`uni.env.USER_DATA_PATH`)

- **用途**：提供给开发者读写用户数据，**不会被系统自动清理**，由开发者自主管理
- **存储位置**：
  - Android: `/sdcard/Android/data/%应用包名%/files/`
  - iOS: `Documents` 目录，内部保留 `uni-app-x` 子目录供真机运行使用

**示例：在用户数据目录写入文件**

```uts
const fs = uni.getFileSystemManager();
fs.writeFile({
	filePath: `${uni.env.USER_DATA_PATH}/hello.txt`,
	data: 'hello uni-app x!',
	encoding: 'utf-8'
} as WriteFileOptions);
```

---

### 2. 内置应用沙盒目录 (`uni.env.ANDROID_INTERNAL_SANDBOX_PATH`)

- **仅支持Android平台**
- **说明**：无法在系统文件管理器中查看，安全性更高
- **用途**：存放框架内置缓存，例如：
  - image/video 组件的网络图片视频缓存
  - web-view 组件缓存
- **访问限制**：FileSystemManager 目前仅支持只读，如需写入需要开发 uts 插件

---

### 3. 沙盒外目录

- **访问限制**：FileSystemManager API 暂不支持访问沙盒外目录
- **特殊场景**：
  - 保存图片/视频到相册有专门 API：`uni.saveImageToPhotosAlbum`、`uni.saveVideoToPhotosAlbum`
  - 其他沙盒外目录访问需要自定义开发 uts 插件

---

## 三、访问规范总结

1. **推荐使用 `uni.env` 常量获取路径**，不推荐硬编码绝对路径
2. **代码包文件只读**，修改需先复制到沙盒目录
3. **缓存目录仅放临时缓存**，关键数据必须放到用户数据目录
4. **uts 插件开发需要绝对路径时**，使用平台提供的转换 API：
   - Android: `UTSAndroid.convert2AbsFullPath`
   - iOS: `UTSiOS.convert2AbsFullPath`

---

## 四、常见问题：路径大小写敏感性

| 平台 | 代码包文件 | 本地磁盘文件 |
|------|------------|--------------|
| Android | 发布后大小写敏感 | 不敏感 |
| iOS 真机 | - | 大小写敏感 |
| iOS 模拟器 | - | 大小写不敏感 |

> 建议：为了兼容性，处理文件路径时始终按照**大小写敏感**原则处理。