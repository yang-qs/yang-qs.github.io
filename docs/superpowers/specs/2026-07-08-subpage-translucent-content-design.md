# 子页面正文区域半透明化设计

Date: 2026-07-08
Status: Approved

## 背景与目标

首页（`home-cinematic` 布局）已经是沉浸式电影胶片风格的大图 Hero。但点击导航进入子页面（About / Research / Talks & Teaching / Blog / CV）后，顶部只有一条 200px 高的小横幅（`.cinematic-banner`，同样用 `cinematic_image` 渲染），横幅下方的正文区域背景是纯色的 `#f7f2e9`（米白色），一路铺到页尾，用户反馈这块是"完全的白底"，希望有一些透明度，让页面不是分层的纯色块。

确认过范围：只改正文区域背景，顶部 `masthead` 导航栏保持不透明不变。

## 视觉方向

参考首页同款胶片调色的 `hero-boya-tower.jpg`，把它做成整页固定背景（滚动时图片不动），正文内容包在半透明磨砂卡片里浮在上面。具体参数：

- 卡片背景：奶白色 `rgba(247, 242, 233, .8)`（对应现有 `--global-bg-color` 的 rgba 版本），配 `backdrop-filter: blur(8px)`
- 不支持 `backdrop-filter` 的浏览器直接退化为纯 80% 半透明色块（无模糊），无需额外兼容代码，可读性仍有保证
- 复用现有 `_cinematic.scss` 的 `cinematic-layers` mixin（照片调色 `brightness(.62) contrast(1.15) saturate(.75)` + 青色调叠加 + 胶片颗粒 + 暗角），只是把它从"仅 200px 横幅内"的 `position: absolute` 变成铺满视口的 `position: fixed`
- 用 `position: fixed` 的普通定位元素实现"固定背景"，而不是 CSS `background-attachment: fixed` 的旧技巧，因此移动端（含 iOS Safari）不需要额外兼容处理

## 范围

只作用于当前已经设置 `header.cinematic_banner: true` 的页面——即导航栏可达的全部 5 个页面（About / Research / Talks & Teaching / Blog / CV），它们现在都已配置同一张 `hero-boya-tower.jpg`。未设置该字段的页面（404、Terms、Sitemap 等）不受影响，行为不变。

首页 `home-cinematic` 布局不在本次改动范围内（首页本身已经是沉浸式大图，没有"白底"问题）。

## 架构与组件改动

1. **新增固定背景层**：在 `_includes/page__hero.html` 的 `cinematic_banner` 分支里，把渲染出的 `.cinematic-banner`（目前是 200px 高、常规文档流内的 `position: relative` 容器）改为一个 `position: fixed`、铺满视口（`inset: 0`）的层，插入在页面内容之前、`z-index` 低于 `masthead`（`masthead` 现为 `z-index: 20`，不变）。内部仍复用 `cinematic-layers` mixin 渲染照片/调色/颗粒/暗角。
2. **移除横幅的"占位挤开"效果，改为顶部留白**：现有 200px 横幅会把下方内容往下推 200px；改为固定背景后，正文卡片前保留一段 240px 高的"顶部留白"（沿用原横幅高度、略微加高），让用户先看到一段干净的照片，再看到正文卡片压上来。
3. **正文容器加磨砂样式**：给 `_layouts/single.html`（About 用）和 `_layouts/archive.html`（Talks / Blog / Research / CV 用）共用的内容容器（`.page__inner-wrap` 及 archive 页对应的列表容器）新增磨砂卡片样式类，应用上面定的 `rgba(247, 242, 233, .8)` + `blur(8px)` + 小圆角。
4. **`_sass/theme/_cinematic.scss` 调整**：新增固定背景层和磨砂卡片的样式规则；`cinematic-layers` mixin 本身逻辑不变，只是新增一处按 `position: fixed` 调用它的地方。

## 不做的事

- 不改 masthead 导航栏的不透明背景
- 不改首页 `home-cinematic` 布局
- 不给未设置 `cinematic_banner` 的页面新增该字段或改动其外观
- 不引入 JS 做滚动视差或额外交互，纯 CSS 实现
