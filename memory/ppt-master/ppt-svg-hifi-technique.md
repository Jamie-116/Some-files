---
name: ppt-svg-hifi-technique
description: 「蓝本派生」方法论 — 以 HTML 前端代码为视觉蓝本，派生出等视觉质量的 SVG 矢量页面（v2.11.0 确认兼容）
metadata:
  type: reference
  priority: high
  originSessionId: 2516d4e8-73f7-473b-94ca-5d9c38cbc0cb
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

## 核心方法论：蓝本派生

> **蓝本派生**：以 HTML 前端代码为视觉蓝本，派生出等视觉质量的 SVG 矢量页面，再经 finalize / pptx 管线输出 PPTX。蓝本不进入最终产物，仅作设计参照。

```
HTML 蓝本（prototypes/*.html，参照用，不进产物）
        ↓ 视觉转译
SVG 矢量（svg_output/*.svg，原生 DrawingML，全元素可编辑）
        ↓ finalize + pptx
      PPTX
```

2026-06-11 陕西德联项目实践总结。方案 1 确立为"蓝本派生"后，工作流为：先用 ui-ux-pro-max / frontend-design 产出 HTML 高保真蓝本 → 以蓝本为视觉参照 → 在 SVG 中用增强矢量技术绘制等视觉质量的 UI 原型。

---

## 技术栈速查（按视觉层级）

### 1. 阴影与深度
```xml
<filter id="cardShadow">
  <feGaussianBlur in="SourceAlpha" stdDeviation="4"/>
  <feOffset dx="0" dy="2"/>
  <feFlood flood-color="#000000" flood-opacity="0.06"/>
  <feComposite in="SourceAlpha" in2="offsetBlur" operator="in"/>
  <feMerge><feMergeNode/><feMergeNode in="SourceGraphic"/></feMerge>
</filter>
```
- 仅用于浮于内容区之上的面板卡片（原型外壳）
- 正文内容卡片不加阴影 → 保持平级感

### 2. 渐变标题栏
```xml
<linearGradient id="hdrGrad" x1="0" y1="0" x2="1" y2="0">
  <stop offset="0%" stop-color="#0082FF"/><stop offset="100%" stop-color="#0055CC"/>
</linearGradient>
<rect x="38" y="86" width="750" height="36" rx="12" fill="url(#hdrGrad)"/>
```
- 原型外壳顶部统一用渐变标题栏
- 颜色跟随子系统主色（生产调控→蓝、户阀管理→紫）

### 3. App Chrome 与窗口装饰
```xml
<circle cx="55" cy="104" r="5" fill="#FF6B6B"/>
<circle cx="71" cy="104" r="5" fill="#FFB347"/>
<circle cx="87" cy="104" r="5" fill="#4CAF50"/>
<text ...>陕西德联一体化平台 — 智能调控中心</text>
```
- 三个红黄绿圆点 + 居中标题，模拟窗口 chrome
- 立即让原型脱离"方框堆砌"的 AI 感

### 4. KPI 指标卡（border-left 强调）
```xml
<rect x="232" y="142" width="180" height="68" rx="10" fill="#FFFFFF" stroke="#E8ECF0" stroke-width="1"/>
<rect x="232" y="142" width="3" height="68" rx="1.5" fill="#0082FF"/>  <!-- 左侧强调条 -->
<text ... font-size="28" font-weight="800" fill="#0082FF">72.5°C</text>
<text ... font-size="11" fill="#8899AA">正常范围 (60-85°C)</text>
```
- 三层信息：标签(11px灰) → 数值(28px粗体色) → 副文本(10px浅灰)
- 左侧 3px 色条提供视觉锚点

### 5. 状态标签徽章（Pill Badge）
```xml
<rect x="540" y="472" width="40" height="18" rx="9" fill="#FF6B6B"/>
<text x="560" y="485" text-anchor="middle" font-weight="600" fill="#FFFFFF">严重</text>
```
- 圆角矩形 rx=高度的一半 → 药丸形状
- 配色编码：严重=#FF6B6B、警告=#FF8C00、提示=#28E6E6、正常=#E8F5E9绿底+深绿字

### 6. 交替行色表格
```xml
<rect x="66" y="286" width="698" height="30" rx="2" fill="#F0F7FF"/>   <!-- 奇数行 -->
<rect x="66" y="320" width="698" height="30" rx="2" fill="#FFFFFF"/>    <!-- 偶数行 -->
<rect x="66" y="354" width="698" height="30" rx="2" fill="#F0F7FF"/>   <!-- 奇数行 -->
```
- 浅蓝底 tint → 白底 → 浅蓝底 交替
- 告警行用红底 tint（#FFF5F5）
- 行高 28-30px，rx=2-3 轻微圆角

### 7. 图表渐变填充
```xml
<linearGradient id="areaGrad" x1="0" y1="0" x2="0" y2="1">
  <stop offset="0%" stop-color="#0082FF" stop-opacity="0.12"/>
  <stop offset="100%" stop-color="#0082FF" stop-opacity="0.01"/>
</linearGradient>
<path d="..." fill="url(#areaGrad)" stroke="none"/>
<polyline points="..." fill="none" stroke="#0082FF" stroke-width="2" stroke-linecap="round"/>
```
- 面积图：先画 fill path，再叠 stroke polyline
- 多系列用不同颜色 + 虚线区分（实线/5-3虚线/3-3虚线）

### 8. 药丸按钮
```xml
<rect x="592" y="450" width="120" height="38" rx="19" fill="#0082FF"/>
<text x="652" y="474" text-anchor="middle" font-weight="600" fill="#FFFFFF">确认执行</text>

<!-- 描边样式 -->
<rect x="720" y="450" width="100" height="38" rx="19" fill="#F0F7FF" stroke="#0082FF" stroke-width="1"/>
<text x="770" y="474" text-anchor="middle" fill="#0082FF">保存方案</text>
```
- 主按钮：纯色填充 + 白字，rx=高度的一半
- 次按钮：浅底 + 描边 + 色字

---

## 方案进化路径

| 层级 | 方案 | 视觉特征 | PPTX 编辑性 | 外部依赖 |
|------|------|---------|------------|---------|
| 当前 | 方案 1 — SVG 高保真 | 阴影/渐变/徽章/交替行色 | ✅ 全矢量可编辑 | 无 |
| 当前 | 方案 2 — 技能混合 | PNG 截图嵌入 | ❌ 图片不可编辑 | ui-ux-pro-max + frontend-design |
| 预留 | 方案 2+ 渲染自动化 | HTML→PNG 自动截图嵌入 | ❌ | headless-browser 渲染 |
| 预留 | 方案 1+ 组件库 | 预置 SVG 组件模板快速拼装 | ✅ | 无（纯模板复用） |

**核心原则**：方案 1 不引入外部渲染依赖，全部用 SVG 原生矢量能力表达高保真度。方案 2 走技能混合生成真实 UI 截图，作为方案 1 的视觉对照。两者并行不悖，方案 3 双页同时产出。

---

**Why**：将方案 1 从"线框图"升级为"高保真矢量"，消除了低信息附加值页面的存在理由。用户得到的是既可编辑、又视觉接近成品软件界面的原型图。技术栈完全基于 SVG 原生能力，与 PPTX 转换器兼容性 100%。
**How to apply**：Executor 生成原型页时，按本规范依次叠加视觉层级：阴影面板 → 渐变标题栏 → KPI 卡片 → 表格 → 徽章 → 按钮。配色严格沿用 spec_lock.md 色系。
