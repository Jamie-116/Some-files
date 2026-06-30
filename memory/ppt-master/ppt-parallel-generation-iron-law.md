---
name: ppt-parallel-generation-iron-law
description: PPT SVG 并行生成铁律 — spec_lock.md 就位后，必须用 Agent 并行生成，严禁主线程逐页 Write（v2.11.0 更新）
metadata:
  type: feedback
  priority: highest
  originSessionId: 2516d4e8-73f7-473b-94ca-5d9c38cbc0cb
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

2026-06-11 陕西德联项目复盘：主线程逐页 Write 了全部 26 页 SVG，违背了提示词中「阶段 2 必须用并行方式调度」的硬性要求。spec_lock.md 已经消除了串行的必要性，但没有利用它。

**根因**：进入阶段二时，思维惯性是"直接写 SVG 最高效"，忽略了并行调度架构 — 这正是 spec_lock.md 设计的目的：让 N 个 Executor 同时读同一份约束、各自独立产出。

---

## 铁律

**spec_lock.md 定稿后，阶段二严禁主线程逐页 Write SVG。必须用 Agent 工具并行启动多个 Executor，各领 3–5 页独立生成。**

---

## 并行策略

### 批次规划

按总页数 18-25 页，拆为 4-6 个并行批次，每个批次负责 3-5 页。同一条消息中启动所有 Agent：

```
批次 1: Cover×2 + TOC + Ending (4页，结构页，最快)
批次 2: Chapter过渡页 ×5 (5页，模板化，简单)
批次 3: Ch1 内容页 ×3 (独立内容)
批次 4: Ch2 架构页 ×3 (独立内容)
批次 5: Ch3 原型页 ×5 (含方案3双页，最复杂)
批次 6: Ch4 原型页 + Ch5 总结 ×6 (独立内容)
```

### 约束传递

每个 Agent 的 prompt 中必须嵌入：
1. 该批次页码和页面标题
2. 每页的具体内容数据（从 spec_lock.md §VI 提取）
3. 该页的布局/隐喻/图表要求
4. **硬约束摘要**（S1-S5+：禁止全幅rect、禁止emoji、禁止混用tspan、框宽容纳文本、侧边标签横排、LaTeX 公式规范）
5. **全局风格参数**（spec_lock.md 中的 `mode` 和 `visual_style`，v2.10.0+ 必传）
6. 输出文件路径
7. images 路径

### 预期加速

| 方式 | 26 页耗时 |
|------|----------|
| 串行逐页 | 15-20 分钟 |
| 6 批次并行 | **~2-3 分钟**（最慢批次决定） |
| 加速比 | **~6-8x** |

---

## 验证方式

并行批次的 Agent 返回后，主线程逐一验证：
1. 文件已写入
2. S1-S5 硬约束合规（用 grep 批量扫）
3. 页间风格一致（色系、字号、间距）

不合规的单页单独修复，不重新生成整批。

---

## 与其他规则的关系

- [[ppt-template-rules]] — 模板继承铁律
- [[ppt-svg-hard-rules]] — SVG 技术硬约束（S1-S5）
- [[ppt-layout-rules]] — 排版设计约束
- [[ppt-preflight-checklist]] — 前置检查清单（阶段三校验时对照）
- [[ppt-memory-vs-skill-priority]] — **规则优先级铁律**（本规则 > SKILL.md §6/7/8 的串行禁令）

**Why**：spec_lock.md 设计初衷就是让多个 Executor 同时读同一份约束源并独立产出 SVG。主线程逐页写等于把并行流水线退化成了串行作坊，完全浪费了 spec_lock 的收敛价值。实测 3 页并行即可获得 ~2x 加速，26 页全量并行预期 6-8x 加速。
**How to apply**：spec_lock.md 定稿后，立即启动 Agent 并行批次。在同一条消息中同时发送所有 Agent 调用，让它们并发执行。每个 Agent 收到截取自 spec_lock 的专属 prompt（包括该批次页码、内容数据、约束摘要、输出路径）。
