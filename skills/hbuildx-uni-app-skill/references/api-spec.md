# uni-app x API规范总结

## 一、API分类

uni-app x项目的uts代码中支持以下类型的API：

| 分类 | 说明 |
|------|------|
| **uts内置API** | 包括内置对象（如`global`），以及平台专有对象`UTSAndroid`、`UTSiOS` |
| **全局API** | 不需要加`uni.`前缀，如`getApp` |
| **uni内置跨平台API** | 以`uni.xxx`形式调用，提供跨平台统一接口 |
| **uniCloud客户端API** | 以`uniCloud.xxx`形式调用，用于连接uniCloud云服务 |
| **DOM API** | 提供DOM相关能力 |
| **Vue API** | Vue框架相关API |
| **平台原生API** | 可直接调用所有平台原生API：<br>- Android 所有原生API<br>- iOS 所有原生API<br>- Harmony 所有原生API<br>- Web浏览器所有API<br>- 小程序所有API |

### 按功能分类的uni API

uni内置API按功能可分为以下类别：
- 全局
- 基础
- 页面和路由
- element和node
- 界面
- 网络
- 设备
- 媒体
- 文件
- 数据存储
- 画布
- 位置
- 登录与认证
- 广告
- 支付
- 分享
- 推送
- 统计
- 组件上下文对象
- Worker
- DOM
- uniCloud客户端API
- 其他扩展API

---

## 二、原生API调用方式

### 1. 直接调用方式

uni-app x 不限制调用任何平台原生API，可直接在uts代码中导入并使用：

```vue
<script>
	// 导入Android原生类
	import Build from 'android.os.Build';
	export default {
		onLoad() {
			// 直接调用原生API
			console.log(Build.MODEL); 
			// 等价于调用uni跨平台API
			console.log(uni.getSystemInfoSync().deviceModel); 
		}
	}
</script>
```

### 2. 从Kotlin转换到UTS的方法

如果已有Kotlin代码，可通过以下方式转换为UTS：

1. **Kotlin代码示例**：
```kotlin
import android.os.Build

fun getDeviceModel(): String {
    return Build.MODEL
}
```

2. **转换后UTS代码**：
```ts
import Build from 'android.os.Build'

function getDeviceModel(): string {
  return Build.MODEL;
}
```

主要语法差异：
- 导入语句：UTS使用 `import 类名 from '包名'` 格式，需用引号包裹包路径
- 函数定义：UTS使用 `function` 替代 Kotlin 的 `fun`
- 变量定义：UTS推荐使用 `let` 替代 `var`，使用 `const` 替代 `val`

### 3. 最佳实践

- 常用跨平台功能优先使用`uni.xxx`API，简单且跨平台
- 复杂原生能力封装为 `uni_modules` 形式的UTS插件，方便共享和跨平台维护
- iOS在js驱动模式下，uvue页面不支持直接调用Swift API，需封装为UTS插件
- uni-app x 不再支持plus和weex API

---

## 三、Promise支持规范

- uni的异步API默认均支持callback方式调用
- **仅部分API支持Promise**，对于支持Promise的API，会在API文档的返回值描述中明确标注包含`Promise`

---

## 四、生命周期规范

生命周期是特殊的事件钩子，uni-app x 为应用、页面、组件分别提供了不同层级的生命周期：

> 注意：`uni.onXX()` 和 `uni.offXX()` 形式的事件监听API不属于生命周期范畴。

### 分类文档

- **应用生命周期**：[详见应用生命周期规范](./api-lifecycle/application-lifecycle.md)
  - 应用启动、显示、隐藏、错误处理等全局事件

- **页面生命周期**：[详见页面生命周期规范](./api-lifecycle/page-lifecycle.md)
  - 页面加载、显示、隐藏、卸载、下拉刷新、上拉触底等页面事件

- **组件生命周期**：[详见组件生命周期规范](./api-lifecycle/component-lifecycle.md)
  - 组件创建、挂载、更新、卸载等组件级生命周期，同时支持选项式API和组合式API

---

## 五、其他说明

- 绝大多数`uni.xxx`API都是使用UTS开发的，开源在 [uni-api 代码仓库](https://gitcode.com/dcloud/uni-api)
- 插件市场提供大量封装好的UTS插件，可直接使用：[uts插件市场](https://ext.dcloud.net.cn/?cat1=8&type=UpdatedDate)