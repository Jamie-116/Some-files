---
name: ppt-template-rules
description: PPT 模板使用规范 — 模板继承铁律、Template-fill直通路径、hx模板image5全幅规则、beautify工作流
metadata: 
  node_type: memory
  type: feedback
  priority: highest
  originSessionId: 0ba0e676-ab55-4223-baac-3df3bcc22da9
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

模板是 PPT 生成的起点，不是可选参考。错误使用模板是最严重的偏差——会导致整份 PPT 风格偏离。与 [[ppt-svg-hard-rules]]、[[ppt-layout-rules]] 协同作用。

**⚠️ v2.10.0+ 新增**：Template-fill 直通路径（规则 3），可直接复用已有 .pptx 模板填充内容，完全绕过 SVG 生成管线。v2.11.0 新增 beautify-pptx 工作流（规则 4），保持文字不变仅重做布局。

---

## 规则 1：模板继承铁律

**2026-05-28 教训**：指定了 `hj_technology_white` 模板但全部做了自由设计。根因：只读了 `design_spec.md`（文字描述），没读实际 SVG 文件，误判模板结构。

### 强制工作流
1. 模板复制到项目后，**必须逐个打开模板 SVG 文件**（至少 cover/toc/content/ending 各一个），不能仅凭 `design_spec.md` 文字描述
2. `spec_lock.md` 中 **必须写 `page_layouts`**，为每一页指定对应的模板 SVG
3. **默认原则**：全部继承模板结构，仅对用户明确要求的冲突点做覆盖。不要反过来默认自由设计再零星对齐
4. 用户额外要求先与模板 SVG 逐项对照：模板已经满足的保留，真正冲突的才覆盖

---

## 规则 2：hj_technology_white 模板 — image5.png 全幅铺满

**仅适用于 hj_technology_white 模板**。

`image5.png` 在所有使用它的页面（TOC、Chapter 过渡页）中，必须作为**全幅背景**铺满整个页面（width=1280, height=720），而非仅左侧边栏（width=424）。

**Why**：image5.png 左侧蓝色 + 右侧白色构成完整底色。之前按模板 SVG 的 424px 宽度使用导致只显示左侧蓝色窄条，右侧大片空白。

### 规范
| 页面区域 | 文字颜色 | 原因 |
|----------|---------|------|
| 左侧蓝色区域（编号、CHAPTER、"目录"等） | 白色 #FFFFFF | 蓝色底上白色可读 |
| 右侧白色区域（标题、副标题、目录列表） | 深色 #2E2E32 / #595757 | 白色底上深色可读 |

- image5.png 统一使用 `width="1280" height="720" preserveAspectRatio="none"`
- 移除白色底色 `<rect fill="#FFFFFF">`（图片本身已含白色右侧）
- 受影响页面：P02_TOC、所有 Chapter 过渡页

---

## 规则 3：Template-fill 直通路径（v2.10.0 新增）

**全新独立路线**：复用已有 `.pptx` 模板，直接编辑 PPTX 原生文件填充新内容，**完全绕过 SVG 生成管线**，输出仍然是原生可编辑的 `.pptx`。

### 适用场景
- 已有公司/机构的标准 PPTX 模板，只需替换内容
- 不需要重新设计视觉风格，只需要更新文字/数据/图表
- 源内容适合直接映射到模板的 slide layout

### 关键护栏
- **check-plan 护栏**：在填充前标记非文本源页面和过度复用同一源 slide 的问题
- Slide library 现在暴露图表数据，私人部件在 slide 复用时隔离
- 详见 ppt-master 的 `workflows/template-fill-pptx.md`

### 与 SVG 管线的选择
```
有可直接复用的 .pptx 模板 + 不需要改设计？
  └─ 是 → Template-fill（更快，输出原生可编辑）
  └─ 否 → SVG 管线（灵活度更高，但需更多步骤）
```

---

## 规则 4：Beautify 工作流（v2.11.0 新增）

`beautify-pptx` 路线：保持已有 Deck 的文字**一字不改**，继承其调色板/字体作为 truth，**仅重做布局**通过 SVG 管线（1:1 页面映射）。

### 适用场景
- 内容已经写好但排版不好看
- 需要统一视觉风格但不想改任何文字
- 从旧 PPTX 迁移到新的 Visual Style

### 与主管线区别
| | 主 SVG 管线 | Beautify | Template-fill |
|---|---|---|---|
| 文字 | 从源文档生成 | **保持原样** | 从源文档生成 |
| 设计 | 全新设计 | **仅重做布局** | 继承模板 |
| 途经 | SVG → PPTX | SVG → PPTX | 直接编辑 PPTX |

---

## 规则 5：多 Deck PPTX 摄入（v2.11.0 新增）

- 支持多个 `.pptx` 文件作为输入源
- 摄入标准化到 `analysis/` 文件夹作为规范的必读文件夹
- `ppt_to_md` 转录原生图表数据为 Markdown 表格，保留源 Deck 超链接
- `pdf_to_md` 识别 `Figure N |` 管道分隔标题
- 源调色板/排版/标识作为**参考而非约束**（主管线中）

---

**Why**：模板是生成质量的第一道防线。继承铁律防止"方向性错误"（整份 PPT 风格偏离）；image5 规则防止特定模板的已知陷阱；Template-fill 提供了绕过 SVG 管线的快速路径；Beautify 解决"只改排版不改字"的高频需求。四条规则覆盖从零生成→模板复用→排版翻新的全场景。
**How to apply**：项目启动时先读本文件；模板复制后执行规则1的工作流；若使用 hj_technology_white 模板则强制启用规则2；若有可复用 .pptx 模板优先评估规则3；若只改排版用规则4。
