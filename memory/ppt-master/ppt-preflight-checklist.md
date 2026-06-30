---
name: ppt-preflight-checklist
description: PPT 生成前置检查清单 — 汇总所有记忆规则为可逐条打勾的验证项，分启动确认/模板准备/逐页生成/全量校验四步，v2.11.0 更新
metadata: 
  node_type: memory
  type: reference
  priority: highest
  originSessionId: 0ba0e676-ab55-4223-baac-3df3bcc22da9
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

每份 PPT 生成时必须逐条核验。三阶段不可跳过。

**阅读顺序**：生成前先读 [[ppt-template-rules]]（模板准备）→ 生成每页时参考 [[ppt-layout-rules]]（选布局模式+Mode+Visual Style）→ Write 后对照 [[ppt-svg-hard-rules]]（技术合规）→ 最后用本文逐条打勾。

**规则源文件**：本文所有检查项均提取自以下文件，遇到不确定的判断标准时回溯原文：
- [[ppt-template-rules]] → T1–T4, T6–T7, L10
- [[ppt-svg-hard-rules]] → S1–S6, L11–L12
- [[ppt-layout-rules]] → L1–L9, 规则0 Mode+Visual Style
- 设计强度方案 (prompt §设计强度) → T0（v2.10.0+ 由 Mode+Visual Style 替代）

---

## 阶段一：启动确认与模板准备（生成前）

- [ ] **T0** 已向用户确认 **Mode**（6 种论证结构：pyramid/narrative/instructional/showcase/briefing + 自定义）和 **Visual Style**（18 种视觉风格，对应 `references/visual-styles/*.md`）。v2.10.0+ 旧的"三种执行器风格"已废弃，必须使用双轴体系。Mode 默认 `pyramid`（政企方案），发布会/TED 风格选 `showcase`。Visual Style 在 v2.11.0 Step 4 交互式确认页中选择，展现为个性光谱（≥3 候选：safe→shifted→bold）。未确认前禁止进入阶段二。
- [ ] **T0a** 已确认原型图生成模式（若 Visual Style 需要原型页）：**方案 1 蓝本派生**（默认）/ **方案 2 技能混合** / **方案 3 双轨原型**。若 Mode=showcase 且需求原型密集：自动锁定方案 3，确认全功能页原型覆盖率 ≥80%。
- [ ] **T0b** 已确认是否使用 Template-fill 直通路径（[[ppt-template-rules]] 规则3）或 Beautify 工作流（规则4）。若选 Template-fill → 跳过阶段二直接进入 PPTX 编辑；若选 Beautify → 确认文字不改、仅重做布局。
- [ ] **T1** 已逐个打开模板 SVG 文件（至少 cover/toc/content/ending 各一个），非仅读 design_spec.md 文字
- [ ] **T2** `spec_lock.md` 中已写 `page_layouts`，每页指定了对应模板 SVG，且已包含 `mode` 和 `visual_style` 字段
- [ ] **T3** 默认继承模板结构，仅对用户明确要求的冲突点做覆盖
- [ ] **T4** 若使用 hj_technology_white 模板：image5.png 确认设为全幅 1280×720
- [ ] **T5** spec_lock.md §VII 中，每页已标注 `chart_source`（`template:<id>` / `template-ref:<id>` / `blueprint`）。对 template / template-ref 页，已列出必提取结构层清单（L1-L5）
- [ ] **T6** 封面管线：已确认具体的 **hook + composition**（v2.11.0+ 不再接受通用标题页，Strategist 必须提交具体的封面策略）
- [ ] **T7** 结尾管线：已确认 **final-impression takeaway**（v2.11.0+ 结尾页必须有明确的收束信息，不能是空洞的"谢谢"页）
- [ ] **T8** 生成模式确认（v2.11.0+ 新增）：spec_lock 定稿后、进入 Phase B 前，已向用户确认串行/并行模式。若 spec_lock 含结构页精确坐标 + page_charts 规格 → 推荐并行；若 spec_lock 粗略或页数 ≤8 → 推荐串行。最终以用户选择为准。参见 [[ppt-v2110-parallel-still-valid]] 规则 7。

---

## 阶段二：逐页生成（每页 SVG Write 后立即执行）

### SVG 技术合规
- [ ] **S1** 无全幅纯色背景 `<rect>`（白色背景不写 rect；图片背景只写 image；遮罩叠在 image 之上）
- [ ] **S2** 无 emoji 字符（全部图标使用 `<use data-icon="...">` 引用图标库）
- [ ] **S3** 同一 `<text>` 内无混用不同 font-weight 的 `<tspan>`（已拆为独立并列 `<text>` 元素）
- [ ] **S4** 无混用不同 font-weight 的 `<tspan>` 配合 `dy` 偏移（同格式 tspan 用 dy 可接受，但竖排仍不推荐；优先独立 `<text>` + 固定 y）
- [ ] **S7** 无 HTML 命名实体（`&ldquo;` `&rarr;` `&middot;` `&bull;` `&hellip;` 等），已用 Unicode 直接字符替代。验证：`grep -Pn '&[a-z]+;' *.svg`（排除预定义5个实体）无输出（[[ppt-svg-hard-rules]] 规则 8）
- [ ] **S8** 无 `<defs>` 图标定义 + `<use href="#xxx">` 内部引用，所有图标使用 `data-icon="tabler-outline/name"` 外部引用。验证：`grep '<defs>' *.svg` 无输出或仅含 `<linearGradient>`（[[ppt-svg-hard-rules]] 规则 9）
- [ ] **S9** XML 属性间有空格（`x="560" y="180"` 而非 `x="560"y="180"`）。验证：`xmllint --noout *.svg` 全部通过（[[ppt-svg-hard-rules]] 规则 10）
- [ ] **S10** 元素间距检查：图标与相邻文字间距 ≥8px，卡片 x 间距 ≥14px y 间距 ≥12px，无 text-anchor 冲突导致的标签重叠（[[ppt-svg-hard-rules]] 规则 11）

### 框宽与溢出
- [ ] **S5** 已执行框宽验证：找出最长文本行 → 计算终点 x = 文本起点 + 字符数 × 字号 × 0.85 → 确认终点 < 框右边界
- [ ] **S6** 若有溢出 → 已裁字/拆行/加宽框，三选一修复

### 排版密度
- [ ] **L1** 空白扫描：页面四边和底部无 ≥80px 连续空白区
- [ ] **L1a** 内容区内部空白扫描：栏间无 ≥60px 连续垂直空白走廊；上下内容组间无 ≥100px 连续水平空白带；卡片内连续空白 ≤ 卡片面积 35%；任意内部空白块 ≤ 150×100px
- [ ] **L2** 拥挤扫描：文字距框边界 ≥8px
- [ ] **L3** 覆盖率估算：所有 `<rect>`/`<image>`/`<text>` 非装饰元素的包围盒总面积 ÷ (1280×720) ≈ 75%–90%。快速算法：目视页面四象限，每个象限都有实质性内容（非纯空白/纯装饰）→ 覆盖率即达标；任一象限完全空白 → 不达标
- [ ] **L4** 内容偏少 → 已用轻量化元素补齐（非放大字号/重复内容/纯装饰色块）
- [ ] **L5** 内容过多 → 已精简文字/压缩间距/拆页（非字号<11px）

### 特殊页面
- [ ] **L6** 若为 TOC 页：采用三行独立 `<text>` 设计（编号22px蓝 + 标题22px深 + 副标题14px灰），章间距 ~80px
- [ ] **L7** 若含 KPI 卡片：Hero Number 字号 ≤ 卡片高度 30%；数字视觉顶部 ≥ 卡片上沿 + 10px；底部 ≥ 标题 + 14px

### 风格一致性
- [ ] **L8** 页面逻辑类型已匹配布局模式 A–F（[[ppt-layout-rules]] 规则 2）；自由排版页面已遵守通用视觉规范（规则 3）
- [ ] **L9** 每页正文 ≤250 字；单卡片正文 ≤3 行每行 ≤30 字
- [ ] **L10** 若使用 hj 模板：左侧蓝色区文字白色，右侧白色区文字深色

### 标签与架构
- [ ] **L11** 侧边标签为横排文字，标签块宽度 ≥60px（非竖排逐字堆叠）
- [ ] **L12** 标签文字垂直居中：y = box_y + box_height/2 + font_size×0.35

### 底部闭合与视觉隐喻（规则 6b/6c）
- [ ] **L13** 正文页底部有闭合元素（总结条/数据标签行/关键结论），高度 30–55px，浅主色底
- [ ] **L14** 本页底部无 ≥80px 孤立空白区（闭合条已覆盖）
- [ ] **L15** 若为章节首/尾页：本章至少 1 页使用了图表化隐喻（漏斗/金字塔/中心辐射/箭头链/环形等）
- [ ] **L16** 若为架构图页（模式 D）：所有层使用相同容器样式（虚线边框 `stroke-dasharray="6 4"` + 浅色底色 + 编号色块 + 层名标签），包括顶层应用层和底层设施层。逐层对比验证容器 rect 属性一致（[[ppt-layout-rules]] 规则 6f、[[ppt-anti-patterns]] A2）
- [ ] **L17** 无自动添加的来源行/脚注行（[[ppt-layout-rules]] 规则 6e）。验证：`grep -i '来源\|source:' *.svg` 无输出（除非用户明确要求）

---

## 阶段三：全量校验（所有页面生成后）

- [ ] **C1** 逐页复查阶段二的 S1–S10（SVG 全量合规），特别关注：全幅 rect（封面/结尾）、HTML 实体（`xmllint --noout`）、内部 defs 引用（`grep '<use href'`）、元素重叠（抽样视觉检查）
- [ ] **C2** 逐页复查阶段二的 L1–L5 + L16（排版密度 + 架构一致性），特别关注底部空白 + 内容区内部空白（L1a）+ 架构图所有层容器样式统一（L16）
- [ ] **C3** TOC 页和所有 Chapter 过渡页复查模板规则：坐标逐元素对照模板 SVG，仅替换占位符文字
- [ ] **C4** 所有 KPI 卡片页复查数字溢出 + 标签无重叠（S10）
- [ ] **C5** 所有架构图页复查：侧边标签横排 + 居中 + **所有层容器样式一致**（L16）
- [ ] **C6** page_layouts 克制检查：正文内容页未标记在 page_layouts 中（仅结构页有标记）
- [ ] **C7** 图表模板覆盖率：全篇 ≥4 个不同 chart template，charts_index.json 已被完整检索
- [ ] **C8** 逐一核实每页底部有闭合元素（无 ≥80px 底部空白）
- [ ] **C9** 图表结构层完整性：所有 `template` / `template-ref` 标注页必须包含 L1-L4 四层（图表主体/标注/指标面板/洞察条），template 页不得因定制内容而省略模板原有结构层
- [ ] **C10** Visual Style 一致性（v2.11.0 新增）：全篇色系、字体、形状语言与 spec_lock.md 锁定的 visual_style 一致。逐页对照 visual_style 的特征自检（如 Swiss-minimal 不应出现圆角卡片，soft-rounded 不应出现直角）
- [ ] **C11** Mode 一致性（v2.11.0 新增）：全篇论证结构符合 spec_lock.md 锁定的 mode。pyramid 模式下每页应有清晰的核心断言（1 个核心信息点），showcase 模式禁止大段文字
- [ ] **C12** 封面与结尾（v2.11.0 新增）：封面有具体 hook+composition（非通用标题），结尾有 final-impression takeaway（非空白"谢谢"）
- [ ] **C13** 源摄入质量（v2.11.0 新增）：若使用多 Deck PPTX 摄入，analysis/ 文件夹内容完整；ppt_to_md 转录的图表数据已检查；源调色板仅作参考非硬约束
- [ ] **C14** 批量修改安全（2026-06-25 新增）：若对 SVG 做过批量操作（sed/脚本），已跑 `xmllint --noout *.svg` 全量验证 + 至少抽样 5 页视觉检查 + `grep -c 'data-icon' *.svg` 图标数量无异常下降。禁止用 sed 删除 `<defs>` 块。修复操作优先用 Python `xml.etree` 而非文本替换（[[ppt-anti-patterns]] C1-C3）

---

**使用方式**：T0 在收到 `/ppt-master` 指令后立即执行（确认 Mode + Visual Style）；阶段一在 `spec_lock.md` 定稿前执行；阶段二每页 SVG Write 后立即逐条打勾；阶段三在所有页面完成后执行，作为最终质量门。**生成前必须通读 [[ppt-anti-patterns]] 反模式大全。**

**版本适配**：本清单已适配 ppt-master v2.11.0 + 融通项目反模式复盘。主要变更：T0 从"设计强度方案 A/B/C/D"替换为"Mode + Visual Style 双轴"；新增 T0b、T6、T7、T8；阶段二新增 S7-S10（HTML实体/内部defs/XML属性/元素重叠）+ L16（架构层一致性）+ L17（来源行）；阶段三新增 C10-C14 五项；C1-C5 扩展覆盖新 S 规则。
