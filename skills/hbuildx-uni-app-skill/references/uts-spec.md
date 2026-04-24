# uni-app x UTS 语法规范总结

## 一、UTS 语言特点

UTS（uni type script）是一门跨平台的、高性能的、强类型的现代编程语言，是 uni-app x 的核心编程语言。

### 核心特点

1. **跨平台编译**：可以编译为不同平台的原生语言：
   - Web/小程序：编译为 JavaScript
   - Android：编译为 Kotlin
   - iOS：编译为 Swift
   - 鸿蒙Next：编译为 ArkTS

2. **语法基础**：采用与 TypeScript 基本一致的语法规范，支持绝大部分 ES6 API，降低了学习门槛。

3. **完全强类型**：相比于 TypeScript，UTS 为了编译为原生语言，类型要求更加严格，所有类型在编译时已知，有助于提前验证代码正确性，提升运行性能。

4. **设计优势**：
   - 编译为平台原生语言，没有跨语言通信成本
   - 不需要较重的运行时，对包体积、内存占用、运行性能影响小
   - 支持各平台原生语言的所有特性，可通过条件编译使用平台特有能力

5. **应用场景**：
   - 在 uni-app 中：用于开发原生扩展插件（API 插件和组件插件）
   - 在 uni-app x 中：作为主编程语言，用于开发所有应用逻辑和扩展插件

### 与 TypeScript 的核心差异

| 特性 | TypeScript | UTS |
|------|------------|-----|
| 类型系统 | 结构化类型系统（基于结构判断兼容性） | 名义类型系统（基于类型名称和显式继承关系） |
| 类型检查 | 类型要求不严格 | 完全强类型，编译时检查 |
| undefined | 支持广泛使用 | 不支持，使用 null 替代 |
| any 类型 | 任意类型 | 适配跨端的任意非空类型，使用时仍需类型转换 |
| 对象字面量类型推导 | 自动推导具体结构类型 | 默认推导为 UTSJSONObject，HBuilderX 4.31+ 支持上下文推导 |

## 二、基本语法

### 1. 变量与常量声明

#### 变量定义（let）
```ts
// 完整语法
let [变量名] : [类型] = 值;

// 示例
let str :string = "hello"; // 声明一个字符串变量
str = "hello world"; // 重新赋值
```

#### 常量定义（const）
```ts
// 完整语法
const [变量名] : [类型] = 值;

// 示例
const str :string = "hello"; // 声明一个字符串常量
str = "hello world"; // 报错，不允许重新赋值
```

#### 命名规则
- 变量名称可以包含数字和字母
- 除了下划线 `_` 外，不能包含其他特殊字符，包括空格
- 变量名不能以数字开头
- 类型定义的冒号左右空格不敏感，`let str:string` 和 `let str : string` 都是合法的

> 注意：不推荐使用 `var` 声明变量，存在平台差异。

### 2. 类型自动推导

UTS 支持类型自动推导，在声明变量并初始化字面量时，可以不显式声明类型：

```ts
// 以下两种写法等价
let s1 = "hello"; // 自动推导为 string 类型
let s2 : string = "hello";

let b1 = true; // 自动推导为 boolean 类型
let b2 : boolean = true;
```

**最佳实践**：除了 boolean 和 string，建议对数字、数组等类型显式声明类型，避免不同版本编译器推导差异导致问题。

HBuilderX 4.31+ 增强了类型推导：
- 对象字面量：可以根据上下文推导为 type 定义的类型，不需要手动 `as`
- 函数返回值：自动推导相同类型或可为空类型的返回值（不支持多个不同类型返回值推导）
- 函数参数：自动推导函数参数数量，调用时可以省略未使用的参数

### 3. 方法定义

方法参数和返回值都需要定义类型：

```ts
// 带参数和返回值
function test(score: number): boolean {
	return (score>=60)
}

// 无返回值使用 void
function add(x :string, y :string) :void {
    let z :string = x + " " + y
	console.log(z)
}
```

### 4. Vue Data 类型定义

在选项式开发中，需要特殊处理 data 类型：
- 字面量可以自动推导类型
- 无法推导的场景需要使用 `as` 显式声明
- 所有项必须初始化，哪怕赋值为 `null`
- UTS 联合类型只支持 `| null`

```ts
<script lang="uts">
	export default {
		data() {
			const date = new Date()
			return {
				s1 : "abc", // 根据字面量推导为 string
				n1 : 0 as number, // 显式声明，也可以省略 as
				n4 : null as number | null, // 可为 null 的数字
				year: date.getFullYear() as number, // 需要显式声明
			}
		}
	}
</script>
```

### 5. 类型判断

- `typeof`：判断布尔、数字、字符串、函数基础类型
- `instanceof`：判断对象具体类型
- `Array.isArray()`：判断数组类型

```ts
// typeof 示例
typeof(true) == "boolean"
typeof("abc") == "string"

// 数组判断
const a1 = ["uni-app", "uniCloud", "HBuilder"]
console.log(Array.isArray(a1)) // true
console.log(a1 instanceof Array) // true

// instanceof 示例
let myDate = new Date();
console.log(myDate instanceof Date) // true
```

### 6. 安全调用

对于可为 null 的类型，调用方法或属性时需要使用问号 `?` 进行安全调用：

```ts
const s: string | null = null 
console.log(s?.length) // 安全调用，编译器不会报错
```

### 7. 核心语法约束

UTS 为了跨平台编译，对 TypeScript 做了一些约束：

1. **不支持 undefined**：所有变量必须初始化，使用 `null` 替代 undefined
2. **条件语句必须使用布尔类型**：不支持隐式类型转换和 truthy/falsy 判断，所有条件必须是布尔表达式
3. **对象字面量只能赋值给 type 定义的类型**：不支持赋值给 interface 定义的类型
4. **不支持变量和函数声明提升**：必须先声明后使用
5. **type 定义不支持嵌套对象字面量**：需要将嵌套对象提取为独立 type
6. **不支持条件类型、映射类型、Utility 类型**：需要手动定义等效类型
7. **不支持 `as const` 断言、确定赋值断言**
8. **类不支持索引访问字段**：只能访问已声明的字段，使用点操作符
9. **类型别名只能在顶层作用域定义**：不能在函数内部定义 type

## 三、跨平台编译规则

### 1. 编译目标映射

| 平台 | 编译目标语言 |
|------|--------------|
| Web/小程序 | JavaScript |
| Android | Kotlin |
| iOS | Swift |
| 鸿蒙Next | ArkTS |

### 2. 变量提升平台差异

不同平台变量提升行为存在差异，推荐跨平台项目始终遵循先定义后使用：

| 平台 | 目标语言 | 全局作用域 | 局部作用域 |
|------|----------|------------|------------|
| Android | Kotlin | 支持先使用后定义 | 必须先定义后使用 |
| iOS | Swift | 支持先使用后定义 | 函数支持先使用，变量必须先定义 |
| 鸿蒙 | ArkTS | 支持先使用后定义 | 函数支持先使用，变量必须先定义 |
| Web | TypeScript | 函数支持先使用，变量必须先定义（var 例外） | 函数支持先使用，变量必须先定义（var 例外） |

### 3. 条件编译

支持条件编译处理多平台差异，语法与 uni-app 一致：

```ts
// #ifdef  %PLATFORM%
平台特有的代码
// #endif

// 示例：区分 Android 和 iOS
// #ifdef APP-ANDROID
// Android 平台特有代码
// #endif
```

支持的条件编译标记：
- `APP-ANDROID` - Android 平台
- `APP-IOS` - iOS 平台
- `UNI-APP-X` - uni-app x 项目
- 其他平台标记与 uni-app 一致

### 4. 编译缓存

uni-app x 编译器使用缓存机制优化编译速度：
- 编译结果缓存存在项目 `unpackage` 目录下
- 代码未变动时直接使用缓存，加快编译
- 升级 HBuilderX 后会自动重新编译
- 若代码修改不生效，可尝试清理构建缓存

### 5. 静态资源处理规则

1. **推荐放置位置**：所有静态资源（图片、字体、音视频等）都放置在项目根目录的 `static` 目录，uni_modules 资源放在 `uni_modules/xxx/static` 目录下。该目录会整体复制到编译产物，变量拼接路径也能正常访问。

2. **不推荐方式**：非 static 目录下的静态资源，只有非变量路径能被识别编译，变量路径资源会丢失。

```html
<!-- 推荐 -->
<image src="/static/3.png"/> 

<!-- 不推荐 -->
<image src="./1.png"/> 
<image :src="imga"/> <!-- 变量路径会丢失 -->
```

### 6. 注意事项

1. 编译 Android 时会产生大量临时 kt/class 文件，建议将 `unpackage` 目录加入安全软件白名单，提升编译性能。
2. 所有 uts 插件都会编译为平台原生语言，web 和小程序平台原生语言就是 JavaScript。
3. uni-app x 的 Android 平台会将所有 uts 代码编译为原生 Kotlin，没有 JS 引擎。

## 总结

UTS 是为跨平台原生开发设计的强类型语言，它基于 TypeScript 语法，做了适合跨平台编译的约束，最终编译为各平台原生语言，兼顾了开发效率和原生性能。开发者需要遵循强类型和先定义后使用等规范，通过条件编译处理平台差异，可以获得接近原生的运行体验。