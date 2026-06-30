---
name: ppt-v2110-confirm-step-must-not-skip
description: v2.11.0 升级后严禁跳过 Step 4 八项确认 — 融通 v1 TOC 坐标偏离、卡片平铺、质量下降的根本原因
metadata: 
  node_type: memory
  type: feedback
  priority: highest
  originSessionId: 01ecc873-e1df-490a-ae3b-137845cd2298
  lastUpdated: "2026-06-25"
---

2026-06-25 融通智算底座项目复盘：v2.11.0 升级后跳过 Step 4 确认步骤直接生成 v1，导致三个系统性质量问题。

## 三大故障模式

| # | 故障 | 表现 | 根因 |
|---|------|------|------|
| 1 | **TOC 坐标偏离** | 左侧"目录"文字过高（y=180 而非 y=330）、右侧条目过低（y=320 而非 y=210） | Agent 自行重排坐标而非照搬模板 SVG |
| 2 | **卡片平铺缺图表** | 内容页大量文字卡片罗列，缺少图表/可视化/隐喻 | 未强制 page_charts 图表驱动规格 |
| 3 | **跳过确认步骤** | 用户未看到 Mode/Visual Style/品质级别选择界面 | AI 直接进入生成阶段，跳过了 Step 4 八项确认 |

**三个故障均非并行/串行导致**，而是流程违规导致。

## 强制规则

### 规则 1：Step 4 八项确认是 BLOCKING 步骤

v2.11.0 SKILL.md §4 规定的八项确认（Mode、Visual Style、品质级别、模板选择、原型方案、封面 hook、结尾 takeaway、spec_lock 确认）不可跳过。未完成全部八项确认前，禁止进入 Phase B（SVG 生成）。

### 规则 2：结构页必须照搬模板坐标

Cover/TOC/Chapter/Ending 四类结构页的每个元素（背景图、标题、编号、Logo）的 x/y/w/h/font-size 都必须从模板 SVG 逐字照搬，仅替换占位符文字。**严禁 Agent 自行重排**。

**修复方式**：
- v2.0 spec_lock §III 写入了 TOC 每行的精确坐标（目录 y=330，条目 y=210/320/430/540），Agent prompt 中直接嵌入这些坐标
- 验证：生成后对比模板 SVG 和输出 SVG 的关键 y 坐标

### 规则 3：内容页必须图表驱动

spec_lock §V `page_charts` 表格为强制项，每页必须标注 chart_source 和可视化主体。全篇 ≥4 个不同 chart template。
- `template:<id>` — 从 charts_index.json 选取
- `template-ref:<id>` — 参照 chart template 的结构自行绘制
- `blueprint` — 从蓝本原型 HTML 派生 SVG

### 规则 4：Visual Style 选择必须可视交互

v2.11.0 Step 4 的确认页包含个性光谱（≥3 候选：safe → shifted → bold），用户在此页选择 Visual Style。如果用户 input 中未指定 style，不能自行假定默认值跳过此步。

---

## 与 SKILL.md 的关系

- SKILL.md §4（Step 4 八项确认）定义了标准流程
- 本规则确保该流程在实际执行中不被跳过
- 若用户明确说"跳过确认，用默认值" → 可以跳过，但必须明确告知使用了哪些默认值
- 与 [[ppt-v2110-parallel-still-valid]] 协同：确认步骤完成后才能决定串行/并行

**Why**：融通 v1 的三个故障（TOC 偏离、卡片平铺、跳过确认）全部可追溯到同一个根因——AI 跳过了 Strategist 确认环节直接生成。修复后 v2.0 的输出质量显著提升，证明确认步骤是 PPT 质量的制度性保障，不是可选的过场。
**How to apply**：每次 `/ppt-master` 启动后，Step 1-3 完成后，必须进入 Step 4 八项确认（用 AskUserQuestion 或交互式页面），全部确认后才能进入 Phase B。结构页坐标从模板 SVG 提取后直接写入 spec_lock §III，在 Agent prompt 中重复。
