---
name: ppt-v2110-parallel-still-valid
description: v2.11.0 SKILL.md 禁止sub-agent要求串行，但 spec_lock 含精确坐标+page_charts 时并行仍有效（记忆优先级高于SKILL.md）
metadata: 
  node_type: memory
  type: feedback
  priority: highest
  lastUpdated: 2026-06-25
  originSessionId: 01ecc873-e1df-490a-ae3b-137845cd2298
---

2026-06-25 融通智算底座项目 v2.0 重做。v2.11.0 SKILL.md §6/7/8 明确写 "NO SUB-AGENT SVG GENERATION" + "SEQUENTIAL PAGE GENERATION ONLY"，列为 MANDATORY 全局执行纪律。这与 [[ppt-parallel-generation-iron-law]] 直接冲突。

## 冲突分析

| 来源 | 指令 |
|------|------|
| v2.11.0 SKILL.md §6/7/8 | 禁止 sub-agent，逐页串行，理由"cross-page consistency depends on per-page authoring with full upstream context" |
| [[ppt-parallel-generation-iron-law]] | spec_lock 就位后必须并行，6-8x 加速 |
| [[ppt-memory-vs-skill-priority]] | 记忆规则 > SKILL.md |

## 用户投诉归因（关键）

用户反馈"升级后输出差了很多"，三个具体问题：
1. TOC 坐标偏离（目录过高/条目过低）→ **根因：spec_lock v1 没有结构页精确坐标，Agent 自行重排**
2. 卡片平铺缺图表 → **根因：Standard 品质 + 没强制 page_charts 图表驱动**
3. 跳过确认步骤 → **根因：流程违规，见 [[ppt-v2110-confirm-step-must-not-skip]]**

**三个投诉均非并行/串行导致**。串行不会自动修复坐标（旧版串行也会错），并行也不会自动导致坐标错（只要 spec_lock 含坐标）。

## 正确选择：并行仍有效，但前提加固

按 [[ppt-memory-vs-skill-priority]] 优先级铁律，记忆规则 > SKILL.md。spec_lock v2.0 满足并行条件（page_layouts 精确坐标 + page_charts 图表驱动 + page_rhythm 完备），并行安全。

**v2.11.0 并行的加固前提（必须同时满足）**：
1. spec_lock §III 含结构页（Cover/TOC/Chapter/Ending）**逐元素精确坐标**（x/y/font-size/anchor 全部写死）
2. spec_lock §V 含每页 `page_charts` 图表驱动规格（图表为主体，非卡片）
3. 每个 Agent prompt **直接嵌入**该批页面的精确坐标/chart 规格（不能只让 Agent 读 spec_lock，要在 prompt 里重复关键坐标）
4. 强制 data-journalism/Premium 等风格五要素自查
5. 生成后跑 `svg_quality_checker.py`（v2.11.0 质量门，之前漏了）

## 何时改用串行

- spec_lock 不含结构页精确坐标（Agent 必须读模板自行推断）→ 串行更安全
- 单页高度依赖前页上下文（如连续故事弧）→ 串行
- 用户明确要求"按新版 SKILL.md 串行" → 尊重用户指令（最高优先级）

**Why**：v2.11.0 SKILL.md 的串行禁令是为防止"无 spec_lock 时 Agent 自由发挥导致跨页不一致"。但有了含精确坐标的 spec_lock，并行 Agent 各自照抄坐标，不存在跨页不一致。串行的代价是 22 页 main-agent 手写必然爆 context（SKILL.md 自己也提供 split mode 应对此）。并行 + 精确 spec_lock 是质量与效率的最优解。

**How to apply**：v2.11.0 项目，若 spec_lock 含结构页精确坐标 + page_charts 规格 → 用并行（记忆规则），Agent prompt 嵌入坐标。若 spec_lock 粗略 → 串行或先补全 spec_lock 再并行。与 [[ppt-v2110-confirm-step-must-not-skip]]、[[ppt-parallel-generation-iron-law]] 协同。

---

## 规则 7：生成前必须让用户选择串行/并行（2026-06-25 新增）

**强制规则**：spec_lock.md 定稿后、进入 Phase B（SVG 生成）之前，AI **必须**主动向用户确认执行模式，不得自行决定。

**交互格式**：
```
spec_lock 已就绪，共 N 页。请选择生成模式：
- **串行**：主 Agent 逐页手写，跨页一致性好，但耗时较长（N 页约 15-20 分钟），且长上下文可能触发压缩
- **并行**：多个 Agent 同时生成，约 2-3 分钟完成，但要求 spec_lock 含精确坐标（当前 spec_lock 已满足 / 不满足）
```

**选择依据**：
| 情况 | 推荐 |
|------|------|
| spec_lock 含结构页精确坐标 + page_charts 规格 | 并行（推荐） |
| spec_lock 粗略，坐标不完整 | 串行 |
| 页数 ≤8 | 串行（页数少时并行优势不明显） |
| 用户明确偏好 | 以用户选择为准 |

**注意**：此规则优先级高于 SKILL.md §6/7/8 的串行禁令，也高于 [[ppt-parallel-generation-iron-law]] 的并行铁律——最终决定权在用户。
