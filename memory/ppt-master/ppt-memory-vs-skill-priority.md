---
name: ppt-memory-vs-skill-priority
description: PPT 规则优先级铁律 — 记忆规则 > SKILL.md，解决两者冲突时的决策瘫痪（v2.11.0 更新）
metadata: 
  node_type: memory
  type: feedback
  priority: highest
  originSessionId: fed595a6-66b3-4f21-83ea-830de3bcc5de
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

2026-06-11 陕西德联项目二次复盘。发现多个场景下 SKILL.md 的默认行为与记忆规则冲突，AI 默认选择了 SKILL.md 导致已知错误重复发生。

**2026-06-24 更新**：ppt-master 已升级至 v2.11.0（本地原 v2.8.0）。SKILL.md 结构可能有重大变化，以下冲突点需在升级后重新验证。

---

## 优先级铁律

```
用户明确指令 > 记忆规则（经实际导出踩坑验证） > SKILL.md > 默认行为
```

**核心逻辑**：记忆规则是"经过实际导出踩坑后验证的硬教训"，SKILL.md 是"通用起点"。当模板源文件、技术规范文档与记忆规则冲突时，**以记忆规则为准** — 因为它来自你已经付出过试错成本的场景。

## 已确认的 5 个冲突及正确选择

### 冲突 1：并行生成策略

| 来源 | 指令 | 来源性质 |
|------|------|---------|
| SKILL.md §6/7/8 | "NO sub-agent SVG generation … sequential page by page … Grouped batches FORBIDDEN" | 通用约束（针对无 spec_lock 的旧工作流） |
| [[ppt-parallel-generation-iron-law]] | "spec_lock 就位后，**必须用 Agent 并行**，严禁主线程逐页 Write" | 2026-06-11 陕西德联复盘教训 |

**正确选择**：记忆规则。spec_lock.md 已将全局信息收敛完毕，并行 Agent 各读同一份约束源独立产出，不存在跨页上下文丢失。实测加速 6-8x。

**判断条件**：spec_lock.md 存在 + page_layouts/page_charts/page_rhythm 完备 → 启用并行。

### 冲突 2：SVG 全幅背景 rect

| 来源 | 指令 | 来源性质 |
|------|------|---------|
| shared-standards.md §4 | "Background: Use `<rect>` to define the page background color" | 通用 SVG 规范 |
| [[ppt-svg-hard-rules]] §S1 | "禁止全幅纯色背景 `<rect>`" | 实际 PPTX 导出踩坑 |

**正确选择**：记忆规则。全幅 `<rect fill="#FFFFFF">` 在 PPTX 中变成可选中/可拖动的多余对象。背景色通过不写任何元素实现（PPTX 默认白底）。

### 冲突 3：模板 image5 尺寸

| 来源 | 指令 | 来源性质 |
|------|------|---------|
| 02_toc.svg / 02a_chapter.svg | `<svg width="424" height="720">` 嵌套 + image5 424px 宽 | 模板原始设计 |
| [[ppt-template-rules]] §2 | "image5 必须全幅 1280×720，而非左侧边栏 424px" | 实际视觉效果验证 |

**正确选择**：记忆规则。image5 左侧蓝色 + 右侧白色构成完整底色，424px 限制只能看到左侧蓝色窄条。

### 冲突 4：页面布局标记策略

| 来源 | 指令 | 来源性质 |
|------|------|---------|
| [[ppt-layout-rules]] §6a（初版） | "正文内容页默认不标记" | 防止整页继承造成千篇一律 |
| [[ppt-layout-rules]] §6a（修正版） | "框架继承 + 内容自由" | 2026-06-11 二次修正 |

**正确选择**：修正版记忆规则。初版矫枉过正导致全篇无 Logo、无模板标题栏。正确做法是标记头部框架（Logo + 标题栏），内容区自由设计。

### 冲突 5：同一 text 内混用 font-weight 的 tspan

| 来源 | 指令 | 来源性质 |
|------|------|---------|
| shared-standards.md §Inline Text Runs | "One logical line with mixed colors/weights/sizes MUST be one `<text>` with inline `<tspan>` children" | 通用 SVG→PPTX 规范（一个 text frame 中转多个 run） |
| [[ppt-svg-hard-rules]] §S3/S4 | "禁止同一 `<text>` 内混用不同 font-weight 的 `<tspan>`" + "禁止混用不同 weight 的 tspan 配合 dy" | 实际 PPTX 导出踩坑 |

**正确选择**：记忆规则。PPTX 转换器对不同 weight 的 tspan 生成独立 `<a:r>`，宽度计算不一致导致实际渲染错行。同一逻辑行内的格式混用**只能混颜色不能混 weight**。需要不同 weight → 拆为独立并列 `<text>` 元素（相同 y 坐标）。

## Agent 决策流程（在启动 PPT 生成前必须执行）

1. **读取全部记忆文件**（`memory/ppt-*.md`）
2. **读取 SKILL.md** + 关联的 reference 文件
3. **逐条对照**以上 4 个冲突点，确认选择
4. **输出选择清单**：`[并行生成: 记忆规则] [全幅rect: 记忆规则] [image5: 记忆规则] [页面标记: 记忆规则修正版]`
5. 在执行过程中，遇到任何 SKILL.md 指令与记忆规则冲突时，**暂停 1 秒思考**："这个 SKILL.md 规则是否已被记忆中的踩坑经验覆盖？"

## 常见"看起来是冲突但实际不冲突"的场景

| 场景 | 解释 |
|------|------|
| SKILL.md 说"禁止子代理"vs 并行生成 | SKILL.md 针对的是无 spec_lock 的旧工作流；有了 spec_lock 后并行是安全且被验证过的 |
| SKILL.md 说"Use rect for background" vs 不写 rect | SKILL.md 说的是"如果需要背景色用 rect"，不适用于"白色背景不需要背景色"的场景 |
| 模板 SVG 的原始坐标 vs 记忆的修正坐标 | 模板是起点，记忆修正来自实际导出验证 |

## v2.10.0+ 潜在新增冲突（升级后需验证）

以下冲突点在 v2.8.0→v2.11.0 升级后可能出现，每次启动 PPT 工作流时需对照 SKILL.md 新版本验证：

### 潜在冲突 6：Mode+Visual Style vs 旧"设计强度方案 A/B/C/D"

| 来源 | 指令 |
|------|------|
| 旧记忆 [[ppt-preflight-checklist]] T0 | "向用户确认设计强度方案：A 专业汇报型 / B 精细方案型 / C 政企精品型 / D 原型驱动型" |
| v2.10.0+ SKILL.md | Mode (pyramid/narrative/instructional/showcase/briefing) + Visual Style (16 种) |

**处理**：旧 A/B/C/D 已废弃，记忆文件已更新为 Mode+Visual Style。若 SKILL.md 仍提及 A/B/C/D，以记忆文件（Mode+Visual Style）为准。

### 潜在冲突 7：Template-fill vs SVG 管线

- Template-fill 是 v2.10.0 新增的旁路路线，当用户提供了可复用的 .pptx 模板时优先使用
- 但若记忆规则要求特定 SVG 质量检查（S1-S5），template-fill 绕过了 SVG 管线，这些检查不适用
- **判断条件**：Template-fill 模式下跳过 S1-S5 检查，改用 check-plan 护栏

### 潜在冲突 8：手绘类风格的路径要求

- v2.11.0 SKILL.md 对于手绘类风格（`sketch-notes`、`chalkboard`、`ink-notes`）要求使用"不规则路径"而非几何原语
- [[ppt-svg-hard-rules]] S1-S5 均基于几何原语，手绘类风格需要不同的验证标准
- **处理**：当 visual_style 为 sketch-notes/chalkboard/ink-notes 时，S4（框宽容纳文本）放宽 15%（不规则路径边界不精确），其余规则维持

**Why**：SKILL.md 是通用技能入口，覆盖所有使用场景（有模板/无模板、有 spec_lock/无 spec_lock）。记忆规则是你在德联、长安天然气等实际项目中反复踩坑后提炼的精准修正。两者冲突时，记忆规则是更细粒度的、针对你具体场景的真理。

**How to apply**：每次启动 PPT Master 工作流时，在读取 SKILL.md 后立即执行本文件 §Agent 决策流程。在生成 spec_lock.md 时，确保以上 4 个冲突点都已按记忆规则的方向选择。
