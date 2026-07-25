# Hugo-Webstack UI 现代化改造计划

## 一、Context

当前项目基于 Hugo + Bootstrap 3.4.1 + jQuery 1.11.1 构建，虽然功能完整，但 UI 设计停留在 2014-2016 年的风格——扁平化、纯色背景、简单的阴影效果、有限的过渡动画。用户希望让界面更加**现代化**、**丝滑流畅**，提升视觉体验。

改造的核心思路是：在**不更换底层框架**（Bootstrap 3、jQuery）的前提下，通过 CSS 自定义属性建立设计系统，利用现代 CSS 特性（`backdrop-filter` 玻璃拟态、多层阴影、过渡动画、CSS 变量）重写样式层，同时优化 JS 交互逻辑，让整个界面焕然一新。

## 二、改造范围

### 涉及修改的文件

| 文件 | 改动程度 | 说明 |
|------|---------|------|
| `themes/hugo-webstack/static/css/nav.css` | **大幅重写** | 建立 CSS 变量体系，现代化所有组件样式 |
| `themes/hugo-webstack/static/js/local-search.js` | 中等 | 搜索框改为 CSS class 切换（配合过渡动画） |
| `themes/hugo-webstack/static/js/nav.js` | 小幅 | 新增卡片入场动画（Intersection Observer） |
| `themes/hugo-webstack/layouts/index.html` | 极小 | 暗色切换微调、移除内联 display:none |
| `themes/hugo-webstack/layouts/search.html` | 极小 | 移除 `#overlay` 的 `style="display:none"` |

### 不变的部分

- Bootstrap 3.4.1 核心 CSS（不修改）
- jQuery 1.11.1 / TweenMax（不修改）
- nav.js 中的侧边栏动画逻辑（保留）
- index.html 中的列宽动态计算 JS（保留）
- Hugo 模板语法（保留）
- `webstack.yml` 数据结构（不变）

## 三、核心设计理念

### 3.1 玻璃拟态（Glassmorphism）

卡片、导航栏、搜索框等容器使用半透明背景 + 背景模糊效果：

```css
background: rgba(255, 255, 255, 0.72);
backdrop-filter: blur(20px);
border: 1px solid rgba(255, 255, 255, 0.3);
```

### 3.2 CSS 变量驱动的设计系统

在 `:root` 中定义完整的颜色、阴影、圆角、间距、过渡变量，统一管理亮色/暗色两套主题。

### 3.3 动画增强

- 所有交互（hover、click、focus）都有平滑的 CSS transition
- 卡片入场使用 Intersection Observer + CSS 动画
- 暗色模式切换有 500ms 的颜色过渡
- 搜索框弹出/关闭使用弹性动画曲线

### 3.4 现代微交互

- 卡片 hover 时：上浮 + 缩放 + 阴影加深 + 顶边光泽
- 侧边栏菜单项 hover 时：左侧出现彩色指示条
- 浮动按钮 hover 时：变为主题色 + 上浮
- 搜索结果项 hover 时：背景高亮 + 平滑过渡

## 四、分阶段实施步骤

### 阶段 1：CSS 基础设施（nav.css 头部）

**目标**：建立完整的 CSS 自定义属性体系

1. 在 `:root` 中定义 ~50 个 CSS 变量：主色调、背景色、文字色、边框色、阴影、圆角、间距、过渡时间、玻璃效果参数、排版、z-index
2. 在 `body.black` 中覆盖暗色模式变量
3. 添加全局基础样式：`body` 过渡、`html { scroll-behavior: smooth }`、自定义滚动条、`::selection`、全局字体
4. 保守处理 `color-mix()` 等较新特性，提供静态回退值

### 阶段 2：侧边栏现代化（nav.css）

1. 背景改为微妙渐变（`linear-gradient`）
2. 菜单项去掉 `border-bottom`，改为圆角 + margin 间距
3. 添加 `::before` 伪元素做左侧彩色指示条（hover/active 时弹簧动画展开）
4. 菜单项 hover 时背景色过渡
5. 保留折叠模式下的图标动画（原有 CSS 逻辑）

### 阶段 3：导航卡片现代化（nav.css）

1. `.box2` 改为玻璃拟态（半透明背景 + `backdrop-filter`）
2. 添加 `::after` 伪元素做顶部光泽条纹
3. hover 时：`translateY(-4px) scale(1.02)` + 多层阴影 + 边框发光
4. 卡片内图片 hover 时缩放旋转
5. `:active` 时按压反馈（缩小 + 快速过渡）
6. 暗色模式下的玻璃效果变体

### 阶段 4：搜索浮层现代化（nav.css + local-search.js）

**CSS 部分**：
1. `#overlay` 改为 `opacity + visibility` 过渡（替代 `display:none` 和 `@keyframes slideDown`）
2. 添加 `backdrop-filter: blur(8px)` 背景模糊
3. `#search-box` 弹出使用弹性动画（`scale(0.95) → scale(1)`）
4. 搜索输入框聚焦时边框发光
5. 搜索结果列表项 hover 高亮优化

**JS 部分**（local-search.js）：
1. 搜索框打开/关闭改用 `classList.add/remove('visible')` 替代 `style.display`
2. Ctrl+F、点击空白处关闭、关闭按钮 同步改为 class 切换
3. 输入框聚焦时自动选中

### 阶段 5：导航栏现代化（nav.css）

1. `.navbar-content` 改为玻璃效果（半透明背景 + 模糊）
2. 导航按钮 hover 时添加背景色过渡
3. 暗色模式适配

### 阶段 6：页脚与浮动按钮（nav.css）

1. 页脚背景改为变量驱动
2. 浮动按钮（回到顶部、搜索、暗色切换）hover 时变主题色 + 上浮 + 阴影
3. 暗色切换图标 hover 时旋转动画

### 阶段 7：卡片入场动画（nav.css + nav.js）

**CSS**：`.xe-card` 初始 `opacity: 0; transform: translateY(20px)`，`.entered` 时显示

**JS**：在 nav.js 末尾添加 Intersection Observer，逐卡片延迟 50ms 触发入场

### 阶段 8：HTML 模板微调

1. `index.html`：暗色切换 JS 增加 localStorage 持久化（可选）
2. `search.html`：移除 `#overlay` 的 `style="display:none"`

### 阶段 9：清理与兼容性

1. 删除 nav.css 中已被变量替代的硬编码颜色规则
2. 保留 PerfectScrollbar、Bootstrap 3 布局、响应式列宽等必须规则
3. 移动端（`max-width: 767px`）降低 `backdrop-filter` 以减少性能开销
4. 确保 `backdrop-filter` 有 `-webkit-` 前缀

## 五、关键技术决策

| 决策 | 方案 | 原因 |
|------|------|------|
| 不升级 Bootstrap | 保持 3.4.1 | 升级会破坏整个布局系统，代价太大 |
| 不升级 jQuery | 保持 1.11.1 | nav.js 和 TweenMax 深度绑定，无法替换 |
| 使用 CSS 变量 | 在 :root 定义 | 无需构建工具，浏览器原生支持，Hugo 直接输出 |
| 玻璃拟态 | backdrop-filter | 现代浏览器广泛支持，移动端降级为不透明 |
| 过渡替代动画 | CSS transition | 比 @keyframes 更适合交互反馈，性能更好 |
| 搜索框改用 class | 替代 display:none | 才能触发 CSS transition 动画 |

## 六、验证方法

1. **本地预览**：运行 `hugo server` 在浏览器中查看效果
2. **亮色/暗色切换**：点击右下角月亮/太阳按钮，确认过渡平滑
3. **卡片 hover**：鼠标悬停卡片，确认上浮 + 缩放 + 阴影 + 光泽效果
4. **卡片点击**：点击卡片，确认按压反馈
5. **搜索框**：Ctrl+F / 点击搜索按钮，确认弹出/关闭动画流畅
6. **侧边栏**：hover 菜单项，确认左侧指示条动画
7. **响应式**：缩放浏览器窗口，确认移动端布局正常
8. **卡片入场**：刷新页面，确认卡片依次淡入上浮
9. **功能回归**：确保所有导航链接可点击、搜索功能正常、侧边栏折叠正常