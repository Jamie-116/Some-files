---
name: ppt-chart-selection-matrix
description: 图表↔模板选型矩阵 + 结构层提取规范 + Executor 双层参照工作流（v2.11.0 更新）
metadata:
  type: reference
  priority: high
  originSessionId: 2516d4e8-73f7-473b-94ca-5d9c38cbc0cb
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

与 [[ppt-chart-blueprint-vs-template]]（对比分析）、[[ppt-svg-hifi-technique]]（蓝本视觉参数）、[[ppt-preflight-checklist]]（T5/C9 检查项）协同使用。

**v2.10.0+ 变更**：Slide library 现在暴露图表数据（charts_index.json 结构已在 v2.7.0 中重构为结构命名——文件名描述视觉结构而非领域模型）。Template-fill 路径中，图表数据可直接映射到 PPTX 原生图表。

---

## 一、图表 ↔ 模板速查

| 页面内容 | 首选模板 | 匹配度 | 策略 |
|---------|---------|--------|------|
| 转化漏斗 / 汇聚流程 | `funnel_chart` | 完全 | 全量骨架 + 改阶段数和数据 |
| 多层技术架构图（4-5 层） | `layered_architecture` | 完全 | 全量骨架 + 改层数和模块卡 |
| 平台/能力中心辐射 | `hub_spoke` | 完全 | 全量骨架 + 改节点内容 |
| 模块功能并列（4-9 个） | `icon_grid` | 完全 | 全量骨架 + 改图标和说明 |
| 子系统组合 | `module_composition` | 完全 | 全量骨架 + 改子系统内容 |
| 垂直时间轴/路线图 | `roadmap_vertical` | 高 | 骨架 + 改阶段节点和状态 |
| 水平时间轴 | `timeline` | 高 | 骨架 + 改里程碑 |
| 功能对比表 | `comparison_table` / `feature_matrix_table` | 高 | 骨架 + 改对比项和列 |
| 4-8 个 KPI 概览卡 | `kpi_cards` | 中 | 提取 KPI 卡片结构 |
| 实施计划表 | `gantt_chart` / `project_schedule_table` | 中 | 骨架 + 改任务/时间/负责人 |
| 双栏对比（优劣/前后） | `pros_cons_chart` | 中 | 提取对比结构 |
| 2×2 象限分析 | `quadrant_text_bullets` / `matrix_2x2` | 中 | 提取象限框架 |
| 循环流程 | `circular_stages` | 中 | 提取环状骨架 |
| 顺序流程 | `process_flow` / `numbered_steps` | 中 | 提取箭头/编号骨架 |
| 词云 | `word_cloud` | 低 | 提取布局 |
| 7→1 异形汇聚 | `funnel_chart`（参照） | 低 | L2-L5 提取，L1 蓝本重建 |
| 定制隐喻（金字塔/中心辐射变体） | 最接近的模板（参照） | 低 | L2-L5 提取，L1 蓝本重建 |
| 原型内嵌图表（曲线/表格/KPI 卡） | 无匹配 | — | 纯蓝本派生 |

## 二、5 层结构提取规范

### L1 — 图表主体
- 提取：模板的核心可视化结构（漏斗 `<path>` 的坐标比例、架构分层 `<rect>` 的排列逻辑、辐射线的角度和 `<line>` 属性）
- 重绘：完全匹配直接改数据；近似匹配需 Executor 按实际内容重建形状，保留模板的布局逻辑（居中、等间距、自上而下等）
- 验证：主体视觉清晰可辨，无重叠、无断层

### L2 — 标注层
- 提取：数据标签 `<text>`、百分比、箭头 `<marker>`、对比数值、连接线 `<line>`
- 重绘：替换为页面实际数据，保持与 L1 主体的空间关系（标注线起点连主体边缘，终点留 8px 间距到文字框）
- 验证：每个数据点均有标注，无孤立无标签的图表段

### L3 — 指标面板（Key Metrics）
- 提取：左侧/底部的 KPI 卡片区块，含色条 `<rect width="4">` + 标题 `<text>` + 数值 `<text font-weight="bold">`
- 重绘：色条颜色映射当前页主色，数值替换为页面实际 KPI
- 验证：至少 2-4 个 KPI 指标，色条与数值色彩一致

### L4 — 洞察条（Insight Bar）
- 提取：图表下方的浅色填充条 `<rect>`，含"关键洞察"/"Key Insight"引导词 + 1-2 行建议文字
- 重绘：底色映射当前页辅助色 10% 透明度，文字替换为页面实际洞察
- 验证：字数 40-80，包含"建议"/"推荐"/"需关注"等行动词

### L5 — 来源脚注（Source Footer）
- 提取：页面底部的灰色小字 `<text fill="#94A3B8"> Source: ...`
- 重绘：替换为页面实际数据来源/统计周期
- 验证：字号 11-12px，颜色 #8899AA 或 #94A3B8

## 三、Executor 双层参照工作流

```
1. 读 spec_lock.md §VII → 获取本页 chart_source
        │
   ┌────┴──────────────────────┐
   │                           │
template / template-ref      blueprint
   │                           │
   ▼                           ▼
2a. 读模板 SVG              2b. 读 HTML 蓝本
   提取 L1-L5 骨架              作为视觉参照
   │                           │
   ▼                           │
3a. 蓝本视觉参数重绘           │
   · 主色→#0082FF              │
   · 阴影 stdDeviation→4        │
   · 按钮 rx→19                │
   · 清除 emoji                │
   · 字体→Microsoft YaHei       │
   │                           │
   └────────┬──────────────────┘
            ▼
4. 填入页面实际数据
   · 图表数据（L1）
   · 标注数值（L2）
   · KPI 指标（L3）
   · 洞察结论（L4）
   · 数据来源（L5）

5. 自检
   - [ ] 5 层结构中 L1-L4 已就位
   - [ ] 配色与 spec_lock.md 色系一致
   - [ ] 无 emoji / 无全幅 rect
   - [ ] 底部闭合条独立于图表洞察条（两者不重叠）
```

## 四、常见反模式

| 反模式 | 正确做法 |
|--------|---------|
| 模板全量拷贝（含 emoji、英文标题、示例数据） | 提取骨架，蓝本重绘，替换全部文字 |
| 标注层全删（"为了简洁"） | 标注是信息厚度来源，必须保留 |
| 图表洞察条与底部闭合条混淆 | 两者独立：洞察条在图表内部 y>600，闭合条在页面最底部 y=678 |
| blueprint 页无结构层 | 纯蓝本派生也需自建 L3（指标面板）和 L4（洞察条）——不能只是一个裸图表 |
| 模板匹配度低但硬套 | 匹配度"低"→ 选 template-ref 而非 template，L1 蓝本重建 |
| 图表页塞多个信息点（v2.10.0+） | 每页一个核心信息点——图表只服务一个主论点 |

## 五、v2.10.0+ 图表页内容质量（新增）

- **每页一个核心信息点**：不再把多个论点塞进一张图表页
- **每内容块独立表达注册器**：不默认使用子弹列表，用最适合该信息的自然表达模式
- **Slide library 暴露图表数据**：Template-fill 路径中可将 Markdown 表格数据直接映射到 PPTX 原生图表

---

**Why**：模板提供的是经过验证的信息架构（5 层），蓝本提供的是风格参考。两者分离使用解决了"蓝本信息薄、模板定制弱"的对立，形成互补工作流。
**How to apply**：Strategist 在 spec_lock 阶段完成 chart_source 标注 + 结构层清单；Executor 在生成时严格按 §三 工作流执行双层参照。
