# 按目标框架的特殊处理

四阶段工作流是通用的，但 Phase 3 写代码时不同框架的还原坑很不一样。这份文档按使用频率从高到低组织，找到对应章节读完再开工。

---

## 微信小程序（WXML + WXSS）

国内最高频场景，坑也最多。**Phase 1 就要把这些前置约束写进 PLAN.md 的"概览"区**，不要留到 Phase 3 才发现。

### 单位换算

WXSS 里用 `rpx`，与 px 的换算关系：

| 设计稿宽度 | rpx 换算 |
|---|---|
| **375**（最常见的设计基准） | 1px = 2rpx |
| 750 | 1px = 1rpx |
| 414 | 1px ≈ 1.81rpx（不推荐，建议设计师改用 375） |

设计稿是 375 宽时，看到 16px 字号 → 写 32rpx。**字号也用 rpx，不要混用 px**。

例外：1px 边框可以保留 `1px`，避免 0.5px 在某些机型变虚。

### 安全区适配

iPhone X 系列以上、刘海屏 Android 都需要避开顶部状态栏和底部 home indicator：

```css
/* 顶部安全区（自定义 navigationBar 时） */
padding-top: env(safe-area-inset-top);

/* 底部安全区（自定义 tabBar 或贴底固定元素时） */
padding-bottom: env(safe-area-inset-bottom);
```

如果设计稿里没体现安全区，**主动问用户**："这个页面顶部/底部有自定义导航/Tab 吗？需要适配安全区吗？"——不要默认套上 padding，会和设计稿对不上。

### 标签替换

HTML 思维 → 小程序原生组件：

| HTML | 小程序 | 备注 |
|---|---|---|
| `<div>` | `<view>` | 通用容器 |
| `<span>` | `<text>` | 行内文本，**`<view>` 里直接写文字也行**但有兼容差异 |
| `<img>` | `<image>` | 必填 `mode`（aspectFit/aspectFill/widthFix 最常用） |
| `<a>` | `<navigator>` 或在 `<view>` 上绑 `bindtap` | 没有原生 hover |
| `<button>` | `<button>` | 注意原生有默认样式，要 `button::after { border: none; }` 重置 |
| `<input>` | `<input>` | 类型用 `type="number/digit/idcard/text"` |
| `<ul><li>` | `<view><view>` | 列表没有原生组件 |

### 原生组件层级问题

下面这几个是原生渲染，**永远在最上层**，普通 view 盖不住它们：

- `video`、`live-player`、`map`、`canvas`（默认）、`textarea`（部分情况）

要盖住的话用 `cover-view` / `cover-image`，但样式能力有限（不支持伪类、动画、复杂选择器）。

如果设计稿里有蒙层弹窗叠加在视频/地图上，**Phase 1 就要标注**，避免后期发现盖不住要返工。

### 滚动

| 需求 | 实现 |
|---|---|
| 整页滚动 | 直接放 view，页面默认可滚 |
| 局部纵向滚 | `<scroll-view scroll-y>`，**必须有显式高度** |
| 局部横向滚 | `<scroll-view scroll-x>`，子元素 `display: inline-block` 或 `white-space: nowrap` |
| 列表很长 | 自己分页，或用 `recycle-view` 库（性能差距明显） |

`overflow: scroll` 在小程序里**不会生效**，必须用 `scroll-view`。

### 字体

- **不支持本地字体文件直接 @font-face**——要在 `app.wxss` 里 `wx.loadFontFace()` 加载远程字体
- **设计稿里的 PingFang SC / 思源黑体**：iOS 自带 PingFang，Android 没有。最佳实践是不指定字体，让系统选；或者只指定 `font-weight`
- 字重在 Android 上的支持比 iOS 差，500 经常显示成 400

### 图片资源

- 网络图片直接写 URL
- 本地图片：放进 `images/` 目录，用相对路径
- **不支持 SVG `<image>` 标签直接显示**——需要转成 base64 内联，或用 `<cover-image>`，或转 PNG。复杂图标推荐用 iconfont
- iconfont：用 `font-family` 加载（受上面字体限制），或用 symbol 模式 + 转 base64

### 全局样式 vs 页面样式

- `app.wxss` 是全局，会被所有页面引入
- 页面 `.wxss` 只在该页面生效
- **CSS 变量在小程序中支持有限**，部分基础库版本不支持。token 文件建议直接在 `app.wxss` 里用 class（`.text-primary { color: #1A6B5E }`），而不是用 `var(--color-primary)`

### 工程化建议

- token 文件：`styles/tokens.wxss`，在 `app.wxss` 里 `@import`
- 组件：用小程序原生 Component 或 Skyline，目录结构 `components/<name>/index.{wxml,wxss,js,json}`
- 阶段 3 还原时，**每个组件单独测试一下能否在真机上正常渲染**，模拟器和真机有时候不一样

---

## uni-app / Taro（跨端框架）

uni-app 和 Taro 都是"一份代码编译多端"，但产物如果跑在小程序上，**上面所有小程序约束依然适用**。

额外注意：

### uni-app
- 标签可以用 `view`、`text`、`image`，也可以用部分 HTML 标签（编译时转换），**优先用 `view`/`text` 保证多端一致**
- 单位用 `rpx` 或 `upx`（uni 自己的，等价于 rpx）
- 路由：`uni.navigateTo` / `uni.redirectTo`
- 条件编译：`#ifdef MP-WEIXIN`、`#ifdef H5` 之类，**不同端有差异时一定要用条件编译，不要妄想一份代码完美跑所有端**

### Taro
- React 写法，但底层编译到小程序，**Hooks 和 React 完全一致**
- 标签用 `View`、`Text`、`Image`（首字母大写、来自 `@tarojs/components`）
- 单位用 px，Taro 编译时按设计稿宽度转 rpx（在 `config/index.js` 里配 `designWidth`）
- 文件结构跟 React 一样（`.tsx` + `.scss`）

### 多端兼容性差异

设计稿如果只有一份，跑到不同端会有差异：

| 差异点 | 微信 | 支付宝 | 抖音 | H5 |
|---|---|---|---|---|
| `safe-area-inset-*` | 支持 | 支持 | 支持 | 支持 |
| 自定义字体 | `loadFontFace` | 类似 | 类似 | 直接 @font-face |
| `position: sticky` | 部分支持 | 部分支持 | 不稳定 | 完整支持 |
| `backdrop-filter` | 不支持 | 不支持 | 不支持 | 支持 |
| `gap` 属性 | 较新版本支持 | 较新版本 | 部分支持 | 支持 |

Phase 1 就要问清"这个产物要跑哪些端"，再决定哪些 CSS 特性能用。

---

## Web 主流（React / Vue / Tailwind / 纯 CSS）

这是 skill 的默认假设场景，约束最少。几条小提醒：

### Tailwind
- token 不放 `tokens.css`，而是放 `tailwind.config.js` 的 `theme.extend`
- 间距 token 直接写阶梯（`spacing: { 18: '4.5rem' }`），不要写 `--space-18`
- 颜色用语义命名（`primary` / `surface`）而非纯名（`teal-500` / `gray-50`），方便品牌色切换
- 阶段 3 写组件时，**不要把所有 Tailwind class 一行堆完**，超过 5 个就用 `@apply` 或抽 component class

### CSS Modules / SCSS
- token 文件用 SCSS variables 或 CSS custom properties，看项目惯例
- 类名 BEM 或 modules 自动哈希都行
- Phase 2 的组件文件结构：`Button/index.tsx` + `Button/Button.module.scss`

### 纯 HTML + CSS
- token 用 CSS variables 放 `:root`，引到 base 样式表
- 组件抽不到 JSX，但可以用 HTML template 或自己复制粘贴
- 适合 PoC 或静态页

---

## Flutter

Flutter 不是 DOM，是 Widget 树。还原设计稿时的几条核心映射：

| 设计稿概念 | Flutter Widget |
|---|---|
| 颜色 hex | `Color(0xFF1A6B5E)` （注意前两位是 alpha） |
| 间距 | `SizedBox(height: 16)` 或 `Padding(padding: EdgeInsets.all(16))` |
| Flex 布局 | `Row` / `Column` + `MainAxisAlignment` / `CrossAxisAlignment` |
| Gap | `SizedBox` 之间或者用 `Wrap` 的 `spacing` |
| 圆角 | `BorderRadius.circular(12)` 包在 `Container` 的 `decoration` |
| 阴影 | `BoxShadow` 列表 |
| 字体 | `TextStyle(fontSize: 14, height: 1.57, fontWeight: FontWeight.w500)`（**height 是行高/字号的比值，不是 px**） |

### 行高陷阱

设计稿写 `font-size: 14, line-height: 22`，Flutter 的 `TextStyle.height` = `22 / 14 ≈ 1.57`。**直接写 22 会得到极大的行间距**，必须算比值。

### Token 组织

```dart
// theme/tokens.dart
class AppColors {
  static const primary = Color(0xFF1A6B5E);
  static const surface = Color(0xFFFFFFFF);
}
class AppText {
  static const body = TextStyle(fontSize: 14, height: 1.57, fontWeight: FontWeight.w400);
}
```

或者用 `ThemeData` + `ColorScheme` + `TextTheme`（更标准但模板代码多）。

---

## SwiftUI

SwiftUI 是声明式，View 协议组合。核心映射：

| 设计稿概念 | SwiftUI |
|---|---|
| 颜色 | `Color(red: 0.10, green: 0.42, blue: 0.37)` 或 Asset Catalog 命名色 |
| 间距 | `.padding(16)` 或 `Spacer()` |
| Flex | `HStack` / `VStack` / `ZStack` + `spacing:` |
| 圆角 | `.cornerRadius(12)` 或 `.clipShape(RoundedRectangle(...))` |
| 阴影 | `.shadow(color:, radius:, x:, y:)` |
| 字体 | `.font(.system(size: 14, weight: .medium))` 或 `.font(.body)` |
| 行高 | `.lineSpacing(8)`（**这里是行间距 = line-height - font-size，不是 line-height 本身**） |

### 行高陷阱

设计稿 `font-size: 14, line-height: 22`，SwiftUI 的 `lineSpacing` = `22 - 14 = 8`。和 Flutter 的逻辑都不一样，每次都要换算。

### Dynamic Type 适配

iOS 用户可以系统级调字号。如果设计稿没考虑这一点，**问用户是否要支持 Dynamic Type**：
- 要支持 → 用 `.font(.body)` 等语义字号，不要写死 `system(size:)`
- 不支持 → 写死 size，但要在 PLAN.md 里标注"不适配 Dynamic Type"

### Token 组织

颜色放 `Assets.xcassets` 的 Color Set，自动适配 Light/Dark：

```swift
// Theme.swift
extension Color {
  static let appPrimary = Color("Primary")  // 来自 Assets
  static let appSurface = Color("Surface")
}
```

---

## React Native

跟 Web 的 React 写法接近，但用 React Native 的组件：

- `<View>`、`<Text>`、`<Image>` 替代 div/span/img
- 样式用 `StyleSheet.create`，**没有 className**，没有 cascade
- 单位都是无单位数字（相当于 dp）
- Flexbox 默认 `flexDirection: 'column'`（和 Web 默认相反！）
- 不支持 CSS variables，token 用 JS 对象

### 行高

`lineHeight` 是直接的 px 数（和 Web 一样），不是比值。设计稿 `line-height: 22` 就写 `lineHeight: 22`。

### 平台差异

```js
import { Platform } from 'react-native';
const styles = {
  shadow: Platform.select({
    ios: { shadowColor: '#000', shadowRadius: 4, shadowOpacity: 0.1 },
    android: { elevation: 4 },
  }),
};
```

iOS 阴影和 Android 阴影用完全不同的 API，**Phase 1 要列双端的差异点**。
