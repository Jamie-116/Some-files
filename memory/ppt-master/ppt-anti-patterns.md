---
name: ppt-anti-patterns
description: PPT 生成全链路反模式大全 — 从模板、SVG、工具链、修复操作四个维度汇总所有已知的"不要这样做"，每次生成前必读
metadata: 
  node_type: memory
  type: feedback
  priority: highest
  originSessionId: 01ecc873-e1df-490a-ae3b-137845cd2298
  lastUpdated: "2026-06-25"
  version: "1.0"
---

本文汇总了从融通智算底座、陕西德联、长安天然气、财会等项目的 PPT 生成过程中积累的所有已知错误模式（anti-patterns），按维度分类。生成前通读本文 + 各维度专项规则（[[ppt-svg-hard-rules]]、[[ppt-layout-rules]]），可避免 90% 的已知问题。

**阅读顺序**：本文先读 → 再读专项规则文件。本文的每个反模式都标注了对应的专项规则出处供深挖。

---

## 一、模板/结构页反模式

### A1. Agent 自行重排结构页坐标（已发生 2 次）
**表现**：TOC 页"目录"文字 y=180（应为 y=330）、条目 y=320（应为 y=210）。Chapter 过渡页编号/标签/标题位置与模板不一致。
**根因**：Agent 根据"美观"自行调整坐标，而非逐元素照搬模板 SVG。
**正确做法**：Cover/TOC/Chapter/Ending 四种结构页，每个元素的 x/y/w/h/font-size/text-anchor 全部从模板 SVG 提取，写入 spec_lock §III，在 Agent prompt 中重复。仅替换占位符文字。
**相关规则**：[[ppt-v2110-confirm-step-must-not-skip]] 规则 2、[[ppt-template-rules]]

### A2. 架构图各层容器样式不统一（已发生 1 次）
**表现**：P08 五层架构中 L1-L4 有虚线边框容器 + 编号色块 + 层名 + 组件卡片，但 L5（应用层）只有裸 pill 标签，缺少虚线边框容器。
**根因**：生成时注意力集中在"内容是否正确"上，忽略了"视觉一致性"。Agent 逐层绘制时对顶层使用了不同于其他层的简化模式。
**正确做法**：
- 架构图的每一层（包括顶层应用层、底层基础设施层）必须使用**完全相同的容器样式**：虚线边框（`stroke-dasharray="6 4"`）+ 浅色底色 + 编号色块 + 层名标签
- 生成后逐层对比：层标签、容器边框、底色 fill-opacity、组件卡片样式 → 全部统一
- 应用层若内容为 pill 标签而非组件卡片，pill 放置在与下层卡片相同的 y 偏移位置
**相关规则**：[[ppt-layout-rules]] 规则 2 模式 D

### A3. 内容页缺少图表，退化为卡片平铺（已发生 2 次）
**表现**：内容页大量文字卡片罗列，全篇只用 2 个 chart template，缺少漏斗/金字塔/中心辐射/箭头链等视觉隐喻。
**根因**：spec_lock 未强制 `page_charts` 图表分配，或 Agent prompt 未指定 chart_source。
**正确做法**：
- spec_lock §V 必须写 `page_charts` 表格，每页标注 chart_source 和可视化主体
- 全篇 ≥4 个不同 chart template，每章 ≥1 页图表化隐喻
- Agent prompt 中嵌入该批页面的 chart_source + 可视化主体描述
**相关规则**：[[ppt-layout-rules]] 规则 6c、[[ppt-preflight-checklist]] C7/C9

---

## 二、SVG 技术反模式

### B1. HTML 实体字符（已发生 2 次）
**表现**：SVG 中出现 `&ldquo;` `&rdquo;` `&middot;` `&rarr;` `&bull;` `&hellip;` 等 HTML 命名实体，XML 解析器直接报错。
**根因**：Agent 用 HTML 习惯写 SVG 文本内容，或从 markdown 转录时保留了 HTML 实体。SVG 是 XML，只支持 5 个预定义实体（`&lt;` `&gt;` `&amp;` `&quot;` `&apos;`），其他命名实体不存在。
**正确做法**：
| HTML 实体 | SVG 替代 |
|-----------|---------|
| `&ldquo;` `&rdquo;` | 直接写 Unicode `"` 或 `「」`，或不用引号 |
| `&middot;` | 直接写 Unicode `·` (U+00B7) |
| `&rarr;` | 直接写 Unicode `→` (U+2192) |
| `&bull;` | 直接写 Unicode `•` (U+2022) |
| `&hellip;` | 直接写 Unicode `…` (U+2026) |
- 生成后用 `xmllint --noout file.svg` 验证 XML 合法性
- 或 `grep -n '&[a-z]\+;' *.svg` 扫描所有非标准实体
**相关规则**：[[ppt-svg-hard-rules]] 规则 8（本文档新增）

### B2. `<use href="#ic-xxx">` 内部 defs 引用（已发生 1 次）
**表现**：SVG 用 `<defs>` + `<use href="#ic-xxx">` 引用内联图标定义。`svg_to_pptx.py` 无法解析内部引用，图标区域渲染为空白。
**根因**：svg_to_pptx 的 PPTX 转换器只处理标准 SVG 形状，不解析 `<use>` 的内部 defs 引用。图标路径未展开为实际形状。
**正确做法**：
- 永远使用外部图标引用：`<use data-icon="tabler-outline/globe" fill="#0082FF"/>`
- `data-icon` 属性由 svg_to_pptx 的图标解析器自动展开为实际路径
- 图标库在 spec_lock §I 中锁定（如 tabler-outline）
- 禁止在 SVG 中使用 `<defs>` 定义图标（仅允许用于 `<linearGradient>` 等渐变定义）
**相关规则**：[[ppt-svg-hard-rules]] 规则 9（本文档新增）

### B3. XML 属性语法错误（已发生 2 次）
**表现**：`<text x="560"y="180">`（`"` 和 `y` 之间缺少空格），导致 XML 解析失败。
**根因**：批量 sed 替换时正则表达式未保留属性间的空格。常见于 `sed` 或 `awk` 对 SVG 做文本替换时。
**正确做法**：
- sed 替换后必须跑 `xmllint --noout *.svg` 验证
- 属性间永远用空格分隔：`x="560" y="180"`（不是 `x="560"y="180"`）
- 写 SVG 时保持一致的属性格式：`name="value"` 之间一个空格
- 用 Python 脚本做 SVG 修改（如 `xml.etree.ElementTree`）优于 sed 文本替换
**相关规则**：[[ppt-svg-hard-rules]] 规则 10（本文档新增）

### B4. 元素重叠（图标/文字/图形/卡片）（已发生 4+ 次）
**表现**：图标与文字重叠、卡片内标签与描述换行覆盖、KPI 标签与分类标签叠加、坐标轴标签与图形重叠。
**具体案例**：
- P18 KPI 页：6 个冗余"维度·xxx"标签与 KPI 分类标签 text-anchor 冲突
- P19 风险页：`↑ 影响程度` 坐标轴标签宽 130px 与左边距交叠
- P09/P11：图标 `transform="scale(1.3)"` 后与标题文字重叠
- 多卡片页面：卡片间距 <12px 导致右侧卡片文字溢出到相邻卡片
**根因**：Agent 生成时逐元素放置，未做全局碰撞检测。icon 缩放后未调整相邻文字 x 坐标。
**正确做法**：
- 图标使用 `transform="translate(x,y) scale(s)"` ，相邻文字 x = 图标 x + 图标实际占用宽度 + 8px
- 两个独立标签/label 永远不要放在同一 x/y 坐标（text-anchor="middle" + text-anchor="start" 冲突是高频错误）
- 卡片最小间距 12px，长文本卡片间距放宽到 16px
- 生成后用可视化方式检查重叠（人工肉眼或脚本检测 bounding box 相交）
**相关规则**：[[ppt-svg-hard-rules]] 规则 11（本文档新增）

### B5. 全幅 rect 白色背景（已发生 2+ 次）
**表现**：`<rect x="0" y="0" width="1280" height="720" fill="#FFFFFF"/>` 在 PPTX 中生成多余可选对象。
**根因**：Agent 习惯性地在 SVG 开头画一个全幅背景矩形，或直接照搬模板 SVG 中的全幅 rect。
**正确做法**：白底不写任何背景元素。图片背景只写 `<image>`。半透明遮罩叠在 image 之上是允许的。
**相关规则**：[[ppt-svg-hard-rules]] 规则 1

### B6. emoji 作为内容元素（已发生 2+ 次）
**表现**：💻🤖📚⚙⚠ 等 emoji 在不同 OS/Office 版本中渲染为方框。
**根因**：为省事直接用 emoji 替代图标，或图标库中找不到对应语义时用 emoji 凑合。
**正确做法**：永远用 `<use data-icon="tabler-outline/xxx">` 引用矢量图标。图标库中无精确匹配 → 选语义最接近的替代品。
**相关规则**：[[ppt-svg-hard-rules]] 规则 2

### B7. 同一 `<text>` 内混用不同 font-weight 的 `<tspan>`（已发生 3+ 次）
**表现**：如 `<text><tspan font-weight="bold">01</tspan><tspan>标题</tspan></text>`，PPTX 中文字错行。
**根因**：PPTX 转换器对不同 weight 的 tspan 生成独立 `<a:r>`，宽度计算不一致。
**正确做法**：拆为独立并列 `<text>` 元素，相同 y 坐标。
**相关规则**：[[ppt-svg-hard-rules]] 规则 3

---

## 三、修复操作反模式

### C1. sed 批量修改未经 XML 验证（已发生 2 次）
**表现**：用 sed 做 SVG 文本替换后，引入属性语法错误（`"y="` 缺少空格），或删除了不应删除的元素（如删除 `<defs>` 导致所有图标消失）。
**根因**：sed 是行级文本替换工具，不理解 XML 结构。删除 `<defs>...</defs>` 也会删除 `<use href="#ic-xxx">` 的引用源，导致图标区域全部变空白。
**正确做法**：
- SVG 修改优先用 Python `xml.etree.ElementTree` 或 `lxml`（理解 XML 树结构）
- 如必须用 sed：① 只做简单文本替换不做结构操作 ② 永远不要删除 `<defs>` ③ 完成后跑 `xmllint --noout *.svg`
- 批量删除内容后，至少抽样 5 页做视觉检查（打开 SVG 看渲染效果）
- `grep -c '<use' *.svg` 验证图标数量没有异常下降
**相关规则**：本文档独立规则

### C2. 删除 `<defs>` 块破坏图标引用（已发生 1 次）
**表现**：v1 版本用 sed 删除了 `<defs>...</defs>`，导致 P08/P16/P19/P20 中 `<use href="#ic-xxx">` 的图标全部不可见。
**根因**：sed 删除 defs 块时，认为 `<use href="#ic-xxx">` 是"多余的内部引用"，但删除了 defs 定义后 use 引用悬空。
**正确做法**：
- 永远不要用 sed 删除 `<defs>` 块
- 如需迁移图标：① 先确认图标库名称 ② 替换 `<use href="#ic-xxx">` 为 `<use data-icon="tabler-outline/name">` ③ 然后才能安全删除 `<defs>`
- 迁移后验证：`grep -c 'data-icon' *.svg` 确认所有图标都已迁移到外部引用
**相关规则**：[[ppt-svg-hard-rules]] 规则 9

### C3. 删除注释时残留 HTML 标记（已发生 1 次）
**表现**：用 sed 删除 `<!-- 来源行 -->` 注释后，残留的 HTML 注释语法标记（如 `-->` 孤立的出现在文本中）未清理干净。
**根因**：sed 匹配模式不精确，只删除了注释标签的一部分。
**正确做法**：
- 删除 XML 注释用精确正则：`sed -i '' 's/<!--.*-->//g' file.svg`
- 删除后检查：`grep -n '<!--' *.svg` 和 `grep -n '-->' *.svg` 确保无残留
- 如果注释内含多行文本，用 Python 脚本处理
**相关规则**：本文档独立规则，与 C1 协同

### C4. 使用 `svg_to_pptx.py -o` 覆盖同名文件导致版本丢失（已发生 1 次）

**表现**：用 `-o <project>/xxx_v2.pptx` 显式路径导出，v1 的同名文件被覆盖，且 `-o` 模式不创建 `backup/` 快照，旧版 PPTX 和对应的 SVG 源全部丢失。同时输出位置不一致——有时在项目根目录（`-o` 模式），有时在 `exports/` 下（默认流）。

**根因**：`svg_to_pptx.py` 有两种输出模式：

| 模式 | 命令 | 输出位置 | 版本留存 |
|------|------|---------|---------|
| 默认流 | `svg_to_pptx.py <project>` | `exports/<项目名>_<时间戳>.pptx` | ✅ 时间戳自动归档 |
| 显式路径 | `svg_to_pptx.py <project> -o xxx.pptx` | 指定路径 | ❌ 覆盖同名文件，无 backup |

**正确做法**：
- **永远使用默认流**（不带 `-o`），PPTX 统一进入 `exports/` 目录，时间戳自动归档
- 禁止使用 `-o` 覆盖已有 PPTX 文件
- 如需自定义文件名，导出后手动重命名 `exports/` 下的文件
- 每版 PPTX 必须保留，不可删除历史版本

**验证**：`ls -la <project>/exports/` 应能看到所有历史版本的带时间戳 PPTX 文件。

**相关规则**：本文档独立规则

---

## 四、流程/交互反模式

### D1. 跳过 Step 4 八项确认（已发生 1 次）
**表现**：AI 直接从源文档生成 spec_lock 后进入 SVG 生成，未向用户展示 Mode/Visual Style/品质级别等确认页。
**根因**：AI 认为"默认值够好了，直接生成可以节省时间"，跳过了 SKILL.md §4 的 BLOCKING 确认步骤。
**后果**：事后发现模板偏离、图表密度不足、质量下降，需要全量重做。
**正确做法**：Step 4 八项确认是 BLOCKING 步骤。无论 AI 认为默认值多合理，都必须走完确认流程。
**相关规则**：[[ppt-v2110-confirm-step-must-not-skip]]

### D2. 未确认串行/并行就启动生成（已发生 1 次）
**表现**：AI 自行决定用并行模式生成 22 页，用户事后反馈"新版本禁止并行，你为什么还并行"。
**根因**：记忆规则优先于 SKILL.md 的串行禁令，但用户有知情权和选择权。
**正确做法**：spec_lock 定稿后、Phase B 启动前，向用户展示：① spec_lock 是否含精确坐标 ② 推荐模式（并行/串行）+ 理由 ③ 让用户选择。**最终决定权在用户**。
**相关规则**：[[ppt-v2110-parallel-still-valid]] 规则 7

### D3. 数据新闻风格来源行自动添加（已发生 1 次）
**表现**：data-journalism 风格的五要素中包含"mono 来源行"，AI 自动在全部 14 个内容页底部添加 `来源：项目可研报告 · 2026.06`。
**根因**：AI 将 Visual Style 的特征描述（"每页底部 13px mono 灰字来源行"）当作必须 100% 实现的规格，未考虑用户场景：企业内部分享报告不需要来源标注。
**正确做法**：
- Visual Style 中的"来源行"是风格特征，不是强制内容
- 除非用户在 prompt 中明确说"加来源标注"或"需要数据来源"，否则不添加任何来源/脚注行
- 此规则优先级高于任何 Visual Style 的特征描述
**相关规则**：[[ppt-layout-rules]] 规则 6e

---

## 五、工具链反模式

### E1. Python 版本不兼容（已发生 1 次，但属一次性修复）
**表现**：ppt-master 脚本使用 `str | None` / `list[str]` 等 PEP 604 语法，Python 3.9 不支持。
**根因**：macOS 自带 Python 3.9，ppt-master 要求 ≥3.10（实际 ≥3.12 以确保所有特性可用）。
**正确做法**：
- 创建专用 venv：`python3.12 -m venv ~/.ppt-master/venv`
- 安装依赖：`~/.ppt-master/venv/bin/pip install python-pptx svglib reportlab mammoth markdownify beautifulsoup4 Pillow numpy openpyxl requests lxml flask`
- 所有 `ppt-master` 命令都通过 venv Python 执行
- 在工作区保留 wrapper 脚本 `~/.ppt-master/bin/ppt-master`
**相关规则**：工具链配置（非记忆文件，属一次性操作）

### E2. `svg_to_pptx.py` 不支持 `<use href="#xxx">` 内部引用
**表现**：使用 `<defs>` 定义图标 + `<use href="#ic-xxx">` 引用的 SVG，转换为 PPTX 后图标区域为空白。
**根因**：svg_to_pptx 的 DrawingML 转换器只展开 `data-icon` 属性中的外部图标库引用，不解析 `<use href="#id">` 的内部 defs 引用。
**正确做法**：图标全部使用 `data-icon="tabler-outline/name"` 外部引用。禁止 `<defs>` 中定义图标（渐变除外）。
**相关规则**：同 B2

---

## 检查清单（生成前扫一遍）

```
□ A1: 结构页坐标从模板 SVG 逐元素照搬？（不是 Agent 自行重排）
□ A2: 架构图所有层使用相同容器样式？（虚线边框 + 编号色块 + 层名 + 组件卡片）
□ A3: page_charts 已分配？每章 ≥1 隐喻页？
□ B1: 无 HTML 命名实体？（grep '&[a-z]\+;' *.svg 无输出）
□ B2: 无 <defs> 图标定义 + <use href="#xxx">？
□ B3: 属性间有空格？（grep '"y=' *.svg 无输出）
□ B4: 元素间距 ≥12px？无重叠风险？
□ B5-B7: S1-S3 硬约束合规？
□ C1: 用 Python 脚本而非 sed 做 SVG 修改？
□ C2-C3: 批量修改后验证 xmllint + grep 残留？
□ D1: Step 4 八项确认已完成？
□ D2: 串行/并行已让用户选择？
□ D3: 无自动添加的来源行？
```

**Why**：融通项目 v1 在模板偏离、卡片平铺、跳过确认、HTML 实体、sed 删除 defs、来源行自动添加这 6 个问题上全部踩坑，每个问题单独看都不致命，但叠加导致输出质量系统性下降，被迫全量重做 v2。本文汇总所有已知反模式，生成前通读一遍可避免 90% 的已知问题。

**How to apply**：每次 PPT 生成前，先通读本文的「检查清单」逐条打勾。在 spec_lock 定稿后、Phase B 启动前，对照四/五类反模式做最后检查。与 [[ppt-svg-hard-rules]]、[[ppt-layout-rules]]、[[ppt-preflight-checklist]]、[[ppt-v2110-confirm-step-must-not-skip]] 协同使用。
