# 应用生命周期规范

`App.uvue`是uni-app-x的主组件，所有页面都是在`App.uvue`下进行切换的，是应用入口文件。应用生命周期仅可在`App.uvue`中监听，在页面监听无效。

`App.uvue`仅支持选项式，暂不支持组合式写法。

## 应用生命周期列表

uni-app-x 支持如下应用生命周期函数：

| 钩子函数 | 说明 | 兼容性（Web/微信小程序/Android/iOS/HarmonyOS） |
|---------|------|-----------------------------------------------|
| `onLaunch(options: OnLaunchOptions)` | 监听应用初始化，应用初始化完成时触发，全局只触发一次 | 4.0 / 4.41 / 3.9 / 4.0 / 4.61 |
| `onShow(options: OnShowOptions)` | 监听应用显示，应用启动或从后台进入前台显示时触发 | 4.0 / 4.41 / 3.9 / 4.0 / 4.61 |
| `onHide()` | 监听应用隐藏，应用从前台进入后台时触发 | 4.0 / 4.41 / 3.9 / 4.0 / 4.61 |
| `onExit()` | 监听应用退出 | x / x / 3.9 / x / 4.72 |
| `onError(error: any)` | 错误监听函数，应用发生脚本错误或 API 调用报错时触发 | 4.0 / 4.41 / 4.21 / 4.21 / 4.61 |
| `onLastPageBackPress()` | 最后一个页面按下Android返回键，常用于自定义退出 | x / x / 3.9 / x / 4.71 |
| `onPageNotFound(options: OnPageNotFoundOption)` | 页面不存在监听函数，应用要打开的页面不存在时触发 | 4.0 / 4.41 / x / x / x |
| `onUnhandledRejection()` | 未处理的 Promise 拒绝事件监听函数 | 4.0 / 4.41 / x / x / - |
| `onThemeChange()` | 监听系统主题变化 | x / 4.41 / x / x / x |

> `x` 表示不支持该平台

## 参数说明

### onLaunch / onShow 参数

| 参数名 | 类型 | 必填 | 说明 |
|-------|------|------|------|
| path | string | 是 | 应用启动页面路径 |
| appScheme | string | 否 | 首次启动时的Scheme（仅Android/iOS支持） |
| appLink | string | 否 | 首次启动时的appLink（仅iOS支持） |

## 重要说明

- **应用生命周期仅可在`App.uvue`中监听**，在其它页面监听无效
- 应用启动参数，也可以通过API `uni.getLaunchOptionsSync()`获取
- 由于Android的`uni.exit()`是热退出，此时很多代码逻辑仍然在运行，需要开发者在 `onExit` 生命周期中手动清理事件监听，避免内存泄露
- 在微信小程序下，打开全屏原生窗体触发应用的 `onHide`，关闭时触发 `onShow`。例如：`chooseImage`、`chooseVideo`、`previewImage`、`chooseLocation`、`openLocation`、`scanCode` 等弹出的原生窗体
- 如果应用通过 scheme 或 applink（通用链接）启动，可在 `onShow` 生命周期获取相应参数，不管是首次启动还是后台激活到前台均会触发

## 示例代码

```vue
<script lang="uts">
export default {
  globalData: {
    text: 'global data'
  },
  onLaunch(options) {
    console.log('App Launch', options.path)
    // 可在此处获取 scheme/appLink 参数进行页面跳转
  },
  onShow(options) {
    console.log('App Show')
  },
  onHide() {
    console.log('App Hide')
  },
  // #ifdef APP-ANDROID
  onLastPageBackPress() {
    console.log('App LastPageBackPress')
    let firstBackTime = 0
    if (firstBackTime == 0) {
      uni.showToast({
        title: '再按一次退出应用',
        position: 'bottom',
      })
      firstBackTime = Date.now()
      setTimeout(() => {
        firstBackTime = 0
      }, 2000)
    } else if (Date.now() - firstBackTime < 2000) {
      uni.exit()
    }
  },
  onExit() {
    console.log('App Exit')
    // 清理事件监听避免内存泄露
  },
  // #endif
  onError(err) {
    console.error('App Error', err)
  }
}
</script>
```
