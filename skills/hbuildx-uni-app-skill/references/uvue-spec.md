# uni-app x uvue渲染引擎规范总结

## 一、uvue概述

uvue是uni-app X中的新一代原生渲染引擎，名称中的"u"代表uni，文件后缀为`*.uvue`。它是DCloud参考vue规范，为非web平台提供的兼容实现，web平台仍内置vue.js。

## 二、uvue特点

1. **兼容vue语法**：完全遵循vue单文件组件规范，支持vue3的选项式API和组合式API，降低vue开发者学习成本
2. **原生渲染性能**：抛弃WebView渲染路径，直接使用各平台原生UI组件渲染，比传统Vue更快的初始化速度、更低的内存占用，性能接近原生开发
3. **轻量架构**：基于虚拟DOM，不依赖Vue.js运行时，优化了事件传递和组件生命周期管理
4. **跨平台一致性**：一套代码可编译到Android（Kotlin）、iOS（Swift）、Web/小程序（JavaScript）、鸿蒙Next（ArkTS）多个平台
5. **响应式数据绑定**：保留了vue的数据双向绑定和Diff算法更新机制，减少手动DOM操作
6. **组件化支持**：支持easycom组件规范，简化组件使用，支持自定义组件和原生插件组件

## 三、语法规范

### 1. 文件结构

uvue遵循vue单文件组件（SFC）规范，包含三个平级根节点：
- `<template>`：模板组件区
- `<script>`：脚本逻辑区
- `<style>`：样式区

与HTML不同，这三个标签在uvue中都是一级节点，平级排列。

**示例（组合式API）：**
```vue
<template>
	<view class="content">
		<button @click="buttonClick">{{title}}</button>
	</view>
</template>

<script setup>
  let title = ref("Hello world")
	const buttonClick = () => {
		title.value = "按钮被点了"
	}
	onLoad(() => {
		// 页面加载逻辑
	})
</script>

<style>
	.content {
		width: 750rpx;
	}
</style>
```

### 2. template语法

- 一个页面/组件只能有一个template标签
- template内只能包含组件（内置基础组件、自定义uvue组件、uts原生插件组件），不支持HTML原生标签
- template下可以有多个根组件
- 支持vue指令（v-if、v-for、v-bind、v-on等）

### 3. script语法

- 只能有一个script标签
- `lang`仅支持uts，无论设置为什么都会按uts编译（iOS的js引擎驱动页面会编译为js）
- `setup`属性标识使用组合式API写法，无此属性则为选项式API
- 所有vue公开API和uni-app生命周期自动引入，无需手动import
- 支持组合式和选项式混合使用，但不支持多个script标签分别编写

**选项式API示例：**
```vue
<script>
	export default {
		data() {
			return {
				title: "Hello world"
			}
		},
		onLoad() {
			// 页面加载逻辑
		},
		methods: {
			buttonClick: function () {
				this.title= "按钮被点了"
			}
		}
	}
</script>
```

### 4. 样式语法

- 允许有多个style标签，支持less、scss、stylus等CSS预处理
- App端支持web CSS子集（称为ucss），保证跨平台一致性：
  - 仅支持class选择器，不支持标签选择器、ID选择器、属性选择器等
  - 仅支持flex布局和绝对布局，flex方向默认纵向（与web不同）
  - 样式不继承，父元素样式不影响子元素
- 支持`UTSJSONObject`和`Map`类型绑定样式数据，Android端Map性能更好
- 支持`v-bind()`动态绑定CSS状态
- 深度选择器`:deep()`仅在Web平台有实际意义，App端不需要

## 四、渲染原理

1. **编译阶段**：
   - 编译器将uvue模板和UTS代码编译为对应平台的原生代码（Android → Kotlin，iOS → Swift，Web → JavaScript）
   
2. **运行阶段**：
   1. 启动时，UVUE引擎将template结构描述转换为对应平台的原生UI组件树，在内存中维护一棵类似浏览器规范的DOM树
   2. 数据初始化完成后，渲染为原生UI界面
   3. 当数据发生变更时，通过Diff算法对比虚拟DOM差异，最小化重绘区域，只更新变化的节点
   4. 直接调用平台原生绘图和布局API进行渲染，不经过WebView中转

3. **DOM操作支持**：
   - 支持通过`uni.getElementById`或`this.$refs`获取`UniElement`对象
   - 支持通过DOM API操作已渲染的元素，但不支持动态创建和删除DOM节点
   - 提供`DrawableContext`用于直接在view上绘制自定义内容

## 五、与Vue的差异对比

| 对比项 | uvue（uni-app x） | 传统Vue（uni-app） |
|--------|-------------------|--------------------|
| 文件后缀 | *.uvue | *.vue |
| 开发语言 | UTS（类TypeScript） | JavaScript/TypeScript |
| 渲染引擎 | 自主轻量原生渲染引擎 | Vue.js 运行时 |
| 渲染方式 | 原生组件直接渲染，无WebView | WebView渲染 |
| 编译产物 | 编译为Kotlin/Swift/JS/ArkTS | 编译为JavaScript |
| 性能表现 | 高，接近原生开发 | 中等，依赖JSBridge |
| CSS支持 | App端仅支持CSS子集，仅class选择器，仅flex/绝对布局 | 支持完整CSS规范 |
| 原生API调用 | 通过UTS直接调用原生API | 通过JSBridge间接调用 |
| 依赖 | 不依赖Vue.js运行时 | 依赖Vue.js运行时 |
| 生态兼容性 | 仅兼容uni-app x生态，不支持大多数Vue生态库 | 兼容完整Vue生态 |
| 学习成本 | 较高，需要掌握UTS语法和原生交互 | 较低，符合前端开发者习惯 |
| 适用场景 | 高性能定制化App、原生深度开发 | 快速开发中小型应用 |

## 六、总结

uvue是uni-app X为了实现高性能跨平台原生应用而设计的渲染引擎，它在保留vue开发者熟悉的语法和开发体验的同时，通过编译到原生代码和原生渲染，获得了接近原生的性能表现，是uni-app跨平台方案的新一代升级。

对于需要极致性能和原生深度定制的项目，uvue是更好的选择；而对于追求开发效率和生态兼容性的快速开发项目，传统Vue方案仍然适用。