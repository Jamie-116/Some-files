---
name: ppt-svg-hard-rules
description: PPT SVG 技术硬约束 — 全幅背景rect、emoji、混用tspan、框宽溢出、竖排标签，违反将导致PPTX导出故障（v2.11.0 更新）
metadata: 
  node_type: memory
  type: feedback
  priority: highest
  originSessionId: 0ba0e676-ab55-4223-baac-3df3bcc22da9
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

SVG 生成时必须遵守的技术硬约束。违反任一条都会导致 PPTX 导出后出现可感知的缺陷（多余形状、错行、方框、溢出、排版分裂）。这些不是风格建议，是经过实际导出踩坑后验证的禁区。

---

## 规则 1：禁止全幅纯色背景 `<rect>`

**问题**：`<rect x="0" y="0" width="1280" height="720" fill="#FFFFFF"/>` 在 PPTX 中变成可选中/可拖动的白色矩形对象，用户点击"背景"时会选中它。

**正确做法**：
- 白色背景 → 不写任何背景元素，PPTX 默认白底
- 图片背景 → 只写 `<image>`，不叠 `<rect>` 在下面
- 半透明遮罩 → `<rect fill="#0082FF" fill-opacity="0.3"/>` 叠在 `<image>` 之上是合法的（遮罩效果，非纯色背景）
- 卡片/区块背景 → 非全幅 `<rect>` 不受此限制

**Edge case**：模板 SVG 中即使有白色背景 `<rect>`，不要照抄。

---

## 规则 2：禁止 emoji 作为图标

**问题**：emoji（💻🤖📚⚙⚠）在不同 OS/Office 版本中渲染不一致，可能显示为方框。

**正确做法**：
- 永远用 `<use data-icon="<library>/<icon-name>" .../>` 引用图标库矢量路径
- 图标库中没有完全匹配的 → 选语义最接近的替代品
- 同项目图标库锁定为一种（chunk-filled / tabler-outline / tabler-filled / phosphor-duotone）
- 政企方案汇报中，任何 Unicode 装饰符号（⚠、✓、✗ 等排版符号除外）都不可作为内容元素

---

## 规则 3：禁止同一 `<text>` 内混用不同 `font-weight` 的 `<tspan>`

**问题**：PPTX 转换器对不同 weight 的 `<tspan>` 生成独立 `<a:r>`，宽度计算不一致导致错行。

**正确做法**：
```xml
<!-- 禁止 -->
<text><tspan font-weight="bold">01</tspan><tspan>标题</tspan></text>

<!-- 正确：独立并列 <text>，相同 y 坐标 -->
<text x="500" y="170" font-weight="bold">01</text>
<text x="544" y="170">标题文字</text>
```
- `dy` 属性也容易产生偏移误差，优先用独立 `<text>` + 固定 y 坐标
- 每行纯文本（无格式混用）可以直接用单一 `<text>`

---

## 规则 4：框宽必须容纳文本内容（先算后画）

**优先级最高。同一错误已发生 5 次（P06 → P20 → P20重做 → P04 → P09），不容再犯。**

**强制工作流**：
```
1. 确定文本内容 + 字号 → 计算像素宽度 = 字符数 × 字号 × 0.85
2. 框宽 = 文本起点 x + 像素宽度 + 右侧留白(≥40px)
3. 验证：文本终点 < 框右边界
4. 如果放不下 → 裁字 / 拆行 / 加宽框，三选一
```

**禁止行为**：先画好看的框，再往里面塞文字。

**验证公式**：
```
文本终点 ≈ 文本起点 x + 字符数 × 字号 × 0.85
框右边界 = 框 x + 框 width
若文本终点 > 框右边界 → 溢出，必须修复
```

---

## 规则 5：侧边标签必须横排（PPTX 不支持竖排文本框）

**问题**：`svg_to_pptx.py` 无法将多行 SVG 文本合并为单个 PPTX 竖排文本框，`tspan dy` 或真实换行都会被分裂为独立段落。

**唯一可靠方案**：标签加宽到 60px，文字横排。

```xml
<!-- 正确：横排标签，PPTX 中是一个文本框 -->
<rect x="62" y="85" width="60" height="135" rx="8" fill="#0082FF"/>
<text x="92" y="158" text-anchor="middle" font-size="15" font-weight="bold" fill="#FFFFFF">应用层</text>
```

**垂直居中公式**：`y = box_y + box_height/2 + font_size*0.35`

---

## 规则 6：LaTeX 公式渲染（v2.9.0+ 新增能力）

v2.9.0 新增 `latex_render.py`，支持 4 供应商后备链渲染 LaTeX 公式为图片。

**使用规范**：
- Strategist 在排版确认后写公式清单，`latex_render.py` 按清单渲染
- 4 供应商后备链：CodeCogs → QuickLaTeX → mathpad → Wikimedia
- `--dry-run` 验证不渲染
- DOCX 中的 Word/MathType (OMML) 公式已被自动重写为内联 LaTeX（v2.10.0+）

**注意**：LaTeX 渲染输出为图片，不是 SVG 矢量。因此在 PPTX 中不可编辑文字。

---

## 规则 7：手绘类风格特殊处理（v2.11.0 新增）

当 `visual_style` 为手绘类（`sketch-notes`、`chalkboard`、`ink-notes`）时，风格使用**不规则路径**而非几何原语：

- `<path>` 替代 `<rect>`（有机边框，非直线）
- `<path>` 替代 `<circle>`（不规则圆形）
- 线条允许轻微抖动（sketch-notes/chalkboard）
- 笔画宽度可能有微妙变化（ink-notes）

**对现有规则的影响**：
- S4（框宽溢出）：放宽 15% 容差（不规则路径边界不精确）
- S3（tspan 混用）：依然适用
- S1（全幅 rect）：用手绘路径替代全幅 rect 的手绘边框，但原理相同（不画全幅背景）

**Why**：v2.9.0 LaTeX 渲染填补了技术文档/学术 PPT 的公式需求；v2.11.0 新增 sketch-notes/chalkboard/ink-notes 三种手绘类风格，要求不同的验证维度。

---

## 规则 8：禁用 HTML 命名实体（v1.0 新增）

**问题**：SVG 中出现 `&ldquo;` `&rdquo;` `&middot;` `&rarr;` `&bull;` `&hellip;` 等 HTML 命名实体，XML 解析器直接报错。SVG 是 XML，只支持 5 个预定义实体（`&lt;` `&gt;` `&amp;` `&quot;` `&apos;`）。

**已发生 2 次**（融通 v1 + 修复中）。

**正确做法**：
| HTML 实体 | SVG 替代 |
|-----------|---------|
| `&ldquo;` `&rdquo;` | 直接写 Unicode `"` 或 `「」` |
| `&middot;` | 直接写 Unicode `·` (U+00B7) |
| `&rarr;` | 直接写 Unicode `→` (U+2192) |
| `&bull;` | 直接写 Unicode `•` (U+2022) |
| `&hellip;` | 直接写 Unicode `…` (U+2026) |

**验证**：`xmllint --noout file.svg` 或 `grep -Pn '&[a-z]+;' *.svg`（排除 `&lt;` `&gt;` `&amp;` `&quot;` `&apos;`）

---

## 规则 9：禁止 `<defs>` 定义图标 + `<use href="#xxx">` 内部引用（v1.0 新增）

**问题**：`<use href="#ic-xxx">` 引用 `<defs>` 中定义的图标，`svg_to_pptx.py` 无法解析内部 defs 引用，图标区域在 PPTX 中渲染为空白。

**已发生 1 次**（融通 v1，P08/P16/P19/P20 图标全部丢失，后用 sed 删除 defs 进一步破坏引用）。

**正确做法**：
- 永远使用外部图标引用：`<use data-icon="tabler-outline/globe" fill="#0082FF"/>`
- `data-icon` 属性由 svg_to_pptx 的图标解析器自动展开为实际形状
- 禁止在 SVG 中写 `<defs>` 定义图标（仅允许 `<linearGradient>` 等渐变定义）
- 图标库在 spec_lock §I 中锁定

**修复已有 SVG**：先替换 `<use href="#ic-xxx">` 为 `<use data-icon="tabler-outline/name">`，然后才能删除 `<defs>`。

---

## 规则 10：XML 属性间必须有空格（v1.0 新增）

**问题**：`<text x="560"y="180">`（`"` 和 `y` 之间缺少空格）导致 XML 解析失败。常见于 sed 批量修改后。

**正确做法**：
- 属性间永远用空格分隔：`x="560" y="180"`
- sed 替换后必须跑 `xmllint --noout *.svg`
- 批量修改 SVG 优先用 Python `xml.etree.ElementTree`，非 sed

**验证**：`grep -rn '"[a-z]' *.svg | grep -v ' "'` 检查缺失空格的属性。

---

## 规则 11：元素间距防重叠（v1.0 新增）

**问题**：图标 transform scale 后与相邻文字重叠、卡片内标签覆盖、KPI 标签堆叠、坐标轴标签溢出。

**已发生 4+ 次**（融通 v2，P09/P11/P12/P14/P15/P16/P18/P19/P20 共 9 页有不同程度的元素重叠）。

**正确做法**：
- 图标使用 `transform="translate(x,y) scale(s)"` 后，相邻文字 x = 图标 x + 图标实际占用宽度 + 8px
- 两个独立标签不要放在同一 x/y 坐标（text-anchor 冲突）
- 卡片 y 间距 ≥12px，x 间距 ≥14px
- 长文本（≥15 字符）卡片宽度 ≥ 字符数 × 字号 × 0.95 + 24px
- KPI 页面的分类标签与数据标签不能叠在同一位置

---

**Why**：这 11 条规则源于实际 PPTX 导出后的缺陷反馈。规则 1-7（全幅rect、emoji、混用tspan、框宽不足、竖排标签、LaTeX、手绘风格）是早期积累的基础禁区。规则 8-11（HTML实体、内部defs引用、属性空格、元素重叠）是融通项目 v1→v2 修复过程中发现的新增禁区，均导致过 XML 解析失败或视觉缺陷。每一条都是"导出后才发现"的坑，前置遵守可消除返工。

**How to apply**：每页 SVG Write 后，逐条对照检查。生成前先读此文件 + [[ppt-anti-patterns]]（反模式大全）。
