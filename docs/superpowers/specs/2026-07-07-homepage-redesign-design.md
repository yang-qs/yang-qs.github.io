# 个人主页艺术化重设计

Date: 2026-07-07
Status: Approved

## 背景与目标

当前网站基于 academicpages Jekyll 主题，是标准的学术侧边栏多页面结构（Publications / Talks / Teaching / Blog / CV），视觉上是通用模板痕迹较重的学术站点。目标是重新设计为更具艺术感的个人主页，参考 [pluskid.org](https://pluskid.org/) 的简洁分区思路，但整体基调更有视觉冲击力：进站即是一张沉浸式大图，风格定为"冷调电影胶片"，并融合杂志排版的编辑感。

## 视觉风格

### 首页 Hero — 冷调电影胶片 + 杂志排版融合

- 全屏背景图，可为任意个人照片（如夕阳/街景/生活照），通过统一的 CSS 滤镜层做"电影分级"：
  - 亮度降低、对比度提升、饱和度略降（`brightness(.62) contrast(1.15) saturate(.75)` 量级，需按实际图片微调）
  - 橙青（orange & teal）双色调叠加：暗部/阴影区压青灰冷色，暖色高光（如天空、光源）保留，用 `multiply` 混合的渐变叠加实现
  - 胶片颗粒纹理（SVG feTurbulence noise，`overlay` 混合，低透明度）
  - 暗角（inset box-shadow）
  - 上下遮幅黑边（letterbox bars），强化电影画幅感
- 排版：
  - 姓名主标题用衬线字体（Georgia / Times New Roman 系），大字号，白/米白色，加文字阴影保证在图片上的可读性
  - 标题上方有一行小号衬线全大写 kicker 文字（定位语，如 "Finance × Applied Math"）
  - 标题下方一条细金色分隔线（`#d9b26a` 量级金色），下接副标题（如 "PhD Candidate, UPenn AMCS"）
  - 导航只在右上角一处，衬线全大写小字号，链接：About · Research · Blog · Talks · CV
  - **不要**左上角重复姓名、不要 "Vol." 期刊号标签、不要帧号装饰、不要首页底部重复的板块标签行——这些都是导航/标题的冗余重复，已在设计过程中排除
- 头像/个人照片不放在 Hero 里（会破坏沉浸感），Hero 图本身就是一张有氛围感的照片

### 子页面（About / Research / Blog / Talks & Teaching / CV）— 暖调编辑衬线（方向 L1）

- 背景切换为暖白/米黄纸感底色（`#f7f2e9` 量级）
- 正文衬线字体（Georgia 系），行距宽松（1.8 左右），适合长文本阅读
- 金色细线（呼应首页金色分隔线）作为分节装饰
- 页面顶部保留一条较矮的 banner（同款胶片滤镜处理的小尺寸图片），做首页与子页的视觉呼应，但不影响下方内容阅读
- About 页采用左图右文的杂志布局：左侧头像/近照，右侧简介文字

## 信息架构

首页为**纯登陆页**：只有 Hero 一屏（大图 + 姓名/定位 + 右上角导航），不在首页堆内容列表。点击导航进入对应独立子页面。

导航精简为 5 项：

1. **About** — 简介 + 头像
2. **Research** — 研究方向 + Publications 列表
3. **Blog** — 随笔文章
4. **Talks & Teaching** — 演讲报告记录 + 教学经历（合并为一个页面）
5. **CV** — 完整简历，页面内含联系方式（邮箱/GitHub 等社交图标）

原站点的 Talks、Teaching 两个独立页面合并；Contact 不单独设页，并入 CV 页或页脚图标。

## 图片可配置性

Hero 背景图（以及子页面顶部 banner 图）通过 Jekyll front matter 配置图片路径，滤镜层（分级/颗粒/暗角/遮幅）作为独立的 CSS 效果统一应用，不依赖具体图片内容。用户可以随时替换成任意新照片，无需改动样式代码。

## 技术实现方向

- 基于现有 Jekyll + academicpages 架构改造，不引入新的静态站点框架
- 新增一个 splash 型首页 layout（`_layouts/splash.html` 基础上扩展，或新建 `_layouts/home-cinematic.html`），胶片滤镜效果用纯 CSS 实现（`filter`、`mix-blend-mode`、SVG noise data URI），不依赖额外 JS 库
- 子页面复用/微调现有 `single.html` layout，替换配色变量（暖白底、金色强调色）和正文字体（衬线）
- 顶部导航精简为 5 项，替换现有 `_data/navigation.yml`
- Talks 与 Teaching 内容合并到一个页面/collection 展示

## 未决事项（留给实现阶段）

- 首页 Hero 具体使用哪张照片（讨论中使用的博雅塔夕阳照可作为候选，或后续替换为其他个人照片）
- About 页头像具体裁切与图片素材
- 子页面顶部 banner 图片的具体选择（可与 Hero 复用同一滤镜但换不同照片）
