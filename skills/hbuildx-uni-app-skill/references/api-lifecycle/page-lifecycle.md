# 页面生命周期规范

uni-app x 页面文件后缀为 `.uvue`，每个页面都有独立的生命周期钩子。在 Vue 中，页面也是一种组件，所以页面也同时支持组件生命周期。

## 页面生命周期列表

| 组合式 API | 选项式 API | 兼容性（Web/微信小程序/Android/iOS/Harmony/Harmony(Vapor)） | 描述 |
|-----------|-----------|-------------------------------------------------------------|------|
| onLoad | onLoad | 4.0 / 4.41 / 3.9 / 4.11 / 4.61 / 5.0 | 生命周期回调，监听页面加载。页面加载时触发，一个页面只会调用一次，可以在 onLoad 的参数中获取打开当前页面路径中的参数。 |
| onPageShow | onShow | 4.0 / 4.41 / 3.9 / 4.11 / 4.61 / 5.0 | 生命周期回调，监听页面显示。页面显示/切入前台时触发。 |
| onReady | onReady | 4.0 / 4.41 / 3.9 / 4.11 / 4.61 / 5.0 | 生命周期回调，监听页面初次渲染完成。页面初次渲染完成时触发，一个页面只会调用一次，代表页面已经准备妥当，可以和视图层进行交互。 |
| onPageHide | onHide | 4.0 / 4.41 / 3.9 / 4.11 / 4.61 / 5.0 | 生命周期回调，监听页面隐藏。页面隐藏/切入后台时触发。如 `navigateTo` 或底部 `tab` 切换到其他页面，应用切入后台等。 |
| onUnload | onUnload | 4.0 / 4.41 / 3.9 / 4.11 / 4.61 / 5.0 | 生命周期回调，监听页面卸载。页面卸载时触发，如 `redirectTo` 或 `navigateBack` 到其他页面时。 |
| onPullDownRefresh | onPullDownRefresh | 4.0 / 4.41 / 3.9 / 4.11 / 4.61 / x | 监听用户下拉动作。需要在 `pages.json` 的页面配置中开启 `enablePullDownRefresh`。处理完数据刷新后，需要调用 `uni.stopPullDownRefresh` 停止下拉刷新。 |
| onReachBottom | onReachBottom | 4.0 / 4.41 / 3.9 / 4.11 / 4.61 / x | 页面上拉触底事件的处理函数。可以在 `pages.json` 中设置触发距离 `onReachBottomDistance`。 |
| onPageScroll | onPageScroll | 4.0 / 4.41 / 3.9 / 4.13 / 4.61 / x | 页面滚动触发事件的处理函数，监听用户滑动页面事件。参数 `scrollTop` 为页面在垂直方向已滚动的距离（单位 px）。 |
| onResize | onResize | 4.0 / 4.41 / 3.9 / 4.11 / 4.61 / x | 页面尺寸改变时触发。 |
| onBackPress | onBackPress | 4.0 / x / 3.9 / 4.11 / 4.61 / 5.0 | 监听页面返回。返回 `true` 时阻止页面返回。`onBackPress` 上不可使用 `async`。iOS 端侧滑返回不会触发。 |
| onInit | onInit | x / x / x / x / 4.61 / x | 生命周期回调，监听页面初始化。页面初始化时触发，一个页面只会调用一次，可以在 onInit 的参数中获取打开当前页面路径中的参数。 |
| onShareAppMessage | onShareAppMessage | x / 4.41 / x / x / 4.61 / x | 用户点击右上角转发。监听用户点击页面内转发按钮或右上角菜单"转发"按钮的行为，并自定义转发内容。 |
| onShareTimeline | onShareTimeline | x / 4.41 / x / x / 4.61 / x | 用户点击右上角转发到朋友圈。 |
| onAddToFavorites | onAddToFavorites | x / 4.41 / x / x / 4.61 / x | 用户点击右上角收藏。 |
| onTabItemTap | onTabItemTap | 4.0 / 4.41 / x / x / 4.61 / x | 当前是 tab 页时，点击 tab 时触发。 |
| onNavigationBarButtonTap | onNavigationBarButtonTap | 4.0 / x / x / x / 4.61 / x | 监听原生标题栏按钮点击事件。 |
| onNavigationBarSearchInputChanged | onNavigationBarSearchInputChanged | 4.0 / x / x / x / 4.61 / x | 监听原生标题栏搜索输入框输入内容变化事件。 |
| onNavigationBarSearchInputConfirmed | onNavigationBarSearchInputConfirmed | 4.0 / x / x / x / 4.61 / x | 监听原生标题栏搜索输入框搜索事件，用户点击软键盘上的"搜索"按钮时触发。 |
| onNavigationBarSearchInputClicked | onNavigationBarSearchInputClicked | 4.0 / x / x / x / 4.61 / x | 监听原生标题栏搜索输入框点击事件。 |

> `x` 表示不支持该平台

## 重要钩子说明

### onLoad
页面初始化时触发，此时 DOM 还未构建渲染完毕，ref 和 getElementById 使用同步方式拿不到 DOM（需要等 onReady 或使用异步获取）。

常见用途：
1. 开始联网取数（联网是异步，不影响页面转场动画）
2. 获取页面 URL 传递的参数

```uts
// 在detail页面的onLoad中接收URL中传递的参数
export default {
  data() {
    return {
      post_id: ""
    }
  },
  onLoad(event : OnLoadOptions) {
    this.post_id = event["post_id"] ?? "";
    // 可根据详情页id继续联网请求数据...
  },
}
```

### onPageShow / onShow
onLoad 只在页面创建时触发一次；而当页面隐藏后再恢复显示时，只会触发 onPageShow，不会触发 onLoad。

> 在组合式 API 中，由于和应用生命周期重名，监听页面的显示隐藏改用 `onPageShow` 和 `onPageHide`。

在微信小程序下，关闭弹出的原生窗体也会触发页面的 onShow，打开时触发 onHide。

### onReachBottom
可在 pages.json 里定义具体页面底部的触发距离，比如设为 50，那么滚动页面到距离底部 50px 时，就会触发 onReachBottom 事件。

### onBackPress
- 参数 `options.from` 表示返回来源：
  - `backbutton`: 顶部导航栏左边的返回按钮或 Android 实体返回键
  - `navigateBack`: 返回 API `uni.navigateBack()`
- 返回 `true` 可阻止默认返回行为
- **注意**：`onBackPress` 不能使用 `async`

## 示例代码（组合式 API）

```vue
<template>
  <view>
    <text>{{ title }}</text>
  </view>
</template>

<script setup lang="uts">
let title = ref("Hello uni-app x")

onLoad((options) => {
  console.log('page loaded', options)
  // 获取参数并加载数据
})

onPageShow(() => {
  console.log('page shown')
  // 刷新数据
})

onReady(() => {
  console.log('page ready')
  // 可操作 DOM
})
</script>
```
