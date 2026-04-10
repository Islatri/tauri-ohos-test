# UI STYLE GUIDE

## 0) 设计风格一句话

**清透玻璃质感 + 紧凑信息密度 + 工具型看板布局 + 温暖到冷静的渐变氛围。**

---

## 1) 必须遵守的基础规则

1. 样式布局均使用纯行内 Tailwind CSS 类，不写 `style` 块和外部 CSS，禁止 `@apply`。
2. 禁止使用 emoji 和内联 SVG。图标统一使用 `lucide-vue-next`。
3. 布局保持紧凑、清晰、可扫描，优先信息效率，不做空洞装饰。
4. 动效只做「有意义的反馈」：进入、悬停、加载，不做无目的炫技。
5. 页面默认兼容桌面和移动端，所有主要区域必须可收缩/换行。

---

## 2) 视觉语言（Visual DNA）

### 2.1 背景与氛围

- 使用多段渐变或径向渐变背景，形成「暖 -> 中性 -> 冷」过渡。
- 叠加 1-2 个低透明度模糊色块（blur blob）制造空间层次。
- 背景不应喧宾夺主，主内容对比度必须足够。

推荐模式（可直接复用）：

- 页面底色：`bg-[radial-gradient(circle_at_top_right,_#fef3c7_0%,_#fffbeb_30%,_#eef2ff_65%,_#e0f2fe_100%)]`
- 背景光斑 A：`bg-cyan-200/40 blur-3xl`
- 背景光斑 B：`bg-amber-200/50 blur-3xl`

### 2.2 卡片系统

- 主容器使用「半透明白 + 边框 + 背景模糊 + 柔和大阴影」。
- 子卡片使用纯白或浅色底，边框略深于背景。
- 所有卡片圆角统一在 `rounded-2xl` ~ `rounded-3xl`，不混用尖锐直角。

推荐模式：

- 主面板：`rounded-3xl border border-slate-200/70 bg-white/85 backdrop-blur shadow-[0_30px_100px_-40px_rgba(15,23,42,0.35)]`
- 普通卡片：`rounded-2xl border border-slate-200 bg-white`
- 强调卡片（深色区块）：`rounded-2xl border border-slate-200 bg-slate-900 text-slate-100`

### 2.3 色彩策略

- 主文本：`slate-900`
- 次文本：`slate-600`
- 辅助文本：`slate-500`
- 状态色：
	- 成功：`emerald` 系
	- 失败：`rose` 系
	- 运行中：`amber` 系
	- 默认/中性：`slate` 系

状态色要「低饱和底 + 中高饱和字/边框」，避免刺眼荧光色。

### 2.4 字体与层级

- 标题建议使用有一点科技感的字体族（如 `Space Grotesk` + 中文回退 `Noto Sans SC`）。
- 层级规则：
	- H1：`text-2xl sm:text-3xl font-bold`
	- 区块标题：`text-sm font-semibold`
	- 正文：`text-sm`
	- 标签/元信息：`text-xs uppercase tracking-wide`

---

## 3) 布局原则

### 3.1 页面骨架

- 外层：全屏容器 + 渐变背景 + 适度内边距。
- 中层：`max-w-*` 限宽居中，防止超宽屏阅读断裂。
- 内层：按信息优先级拆分为「总览区 / 列表区 / 辅助区」。

推荐骨架：

1. 顶部 Header：标题、说明、主操作按钮。
2. 指标区 Stats：2~4 个关键数字卡。
3. 主体区 Content：左侧任务主体，右侧说明与日志。

### 3.2 栅格与间距

- 全局间距以 `gap-3` / `gap-4` 为主，保证紧凑但不拥挤。
- 卡片内边距常用 `p-3` / `p-4` / `p-6`。
- 小屏优先堆叠，大屏再切多列：
	- 示例：`grid grid-cols-2 md:grid-cols-4`
	- 示例：`grid gap-3 lg:grid-cols-[1.4fr_1fr]`

---

## 4) 组件规范（可复用模板）

### 4.1 顶部标签（Badge Chip）

- 作用：传达模块身份或页面定位。
- 风格：深底浅字，小尺寸，圆胶囊。
- 模板：`inline-flex items-center gap-2 rounded-full bg-slate-900 px-3 py-1 text-xs font-semibold tracking-wide text-white`

### 4.2 主按钮 / 次按钮

- 主按钮：深色实底，明确主动作。
	- `rounded-xl bg-slate-900 text-white hover:bg-slate-700 disabled:bg-slate-500`
- 次按钮：浅底描边，用于次级动作。
	- `rounded-xl border border-slate-300 bg-white text-slate-700 hover:border-slate-400 hover:bg-slate-50`
- 按钮统一行高和内边距，不要出现多个尺寸体系。

### 4.3 状态徽章（PASS/FAIL/RUNNING/IDLE）

- 统一为「圆胶囊 + ring + 图标 + 文本」。
- 不同状态只换配色，不换结构。

示例映射：

- `pass`: `text-emerald-700 bg-emerald-50 ring-emerald-200`
- `fail`: `text-rose-700 bg-rose-50 ring-rose-200`
- `running`: `text-amber-700 bg-amber-50 ring-amber-200`
- `idle`: `text-slate-600 bg-slate-100 ring-slate-200`

### 4.4 信息卡片（List Card）

- 结构：分类 > 标题 > 说明 > 底部动作行。
- 悬停反馈：轻微上浮 + 阴影增强。
- 模板：`rounded-2xl border border-slate-200 bg-white p-4 transition hover:-translate-y-0.5 hover:shadow-md`

### 4.5 日志区（Console Block）

- 使用深色底形成视觉分区。
- 文字保持小字号 + 高行高，支持滚动。
- 模板：`bg-slate-900 text-slate-100` + `max-h-* overflow-auto` + `text-xs leading-5`

---

## 5) 动效规范

1. 页面首屏：容器从 `translate-y-3 opacity-0` 到 `translate-y-0 opacity-100`。
2. 卡片悬停：仅轻微位移（`-translate-y-0.5`）和阴影增强。
3. 加载状态：图标旋转（`animate-spin`），避免整块闪烁。
4. 所有动画都要短（150~300ms），避免拖沓。

---

## 6) 文案风格

1. 标题短、功能导向，尽量 2~4 个词。
2. 描述句只解释「这个区块做什么」，不写营销文案。
3. 日志文案优先包含动作 + 结果 + 耗时。
4. 中英混排时优先保证技术词准确，不强行完全中文化。

---

## 7) 页面快速搭建清单（Checklist）

每次新页面上线前，至少确认：

1. 是否使用了渐变背景 + 柔光斑层次。
2. 是否采用了玻璃主容器和统一圆角体系。
3. 是否区分了主按钮与次按钮。
4. 是否有明确状态色映射（成功/失败/运行中/默认）。
5. 是否在移动端保证可读（小屏堆叠，大屏分栏）。
6. 是否控制了动效数量和时长。
7. 是否完全遵守「只用 Tailwind 行内类 + lucide 图标」规则。

---

## 8) Do / Don’t

### Do

- 使用 `slate` 作为中性基底，再叠加少量功能色。
- 保持卡片和按钮的统一圆角与边框语言。
- 用轻量动效强化状态，而不是装饰。
- 在同一页面内维持一致的间距节奏（`gap-3/4`, `p-3/4/6`）。

### Don’t

- 不要回到纯白平面背景，页面会失去氛围层级。
- 不要为每个卡片使用不同视觉风格，避免拼贴感。
- 不要使用高饱和荧光色做大面积背景。
- 不要引入 emoji、内联 SVG 或额外 CSS 文件破坏一致性。

---

## 9) 一段可复用的页面起手式（Tailwind 结构参考）

```vue
<main class="relative min-h-screen overflow-hidden bg-[radial-gradient(circle_at_top_right,_#fef3c7_0%,_#fffbeb_30%,_#eef2ff_65%,_#e0f2fe_100%)] px-4 py-6 sm:px-8 sm:py-10">
	<div class="pointer-events-none absolute -left-10 top-8 h-44 w-44 rounded-full bg-cyan-200/40 blur-3xl"></div>
	<div class="pointer-events-none absolute -right-16 top-20 h-56 w-56 rounded-full bg-amber-200/50 blur-3xl"></div>

	<section class="relative mx-auto w-full max-w-6xl rounded-3xl border border-slate-200/70 bg-white/85 p-4 shadow-[0_30px_100px_-40px_rgba(15,23,42,0.35)] backdrop-blur md:p-6">
		<!-- header -->
		<!-- stats -->
		<!-- content -->
	</section>
</main>
```

这段结构可作为新页面脚手架，再按具体业务填充内容。
