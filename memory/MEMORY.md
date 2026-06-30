# 工作区记忆

## 项目概览
工作区路径: <workspace_root>

## 主要项目

## PPT Master 工作流（`memory/ppt-master/`）

所有 PPT 生成相关的记忆规则和布局参照已收敛到 `memory/ppt-master/` 子目录。
**当前适配版本：ppt-master v2.11.0**（2026-06-24 从 v2.8.0 升级，记忆文件已全部更新）。

### 优先级规则
- [PPT 记忆规则 vs SKILL.md 优先级铁律](ppt-master/ppt-memory-vs-skill-priority.md) — **最高优先级，每次启动 PPT 工作流必须先读**。v2.11.0 新增 3 个潜在冲突点（Mode+VS vs 旧A/B/C/D、Template-fill vs SVG管线、手绘风格路径要求）
- [PPT 反模式大全](ppt-master/ppt-anti-patterns.md) — **生成前必读**，汇总模板/结构页、SVG技术、修复操作、流程交互、工具链五类共 17 个已知错误模式 + 生成前检查清单（融通项目 v1→v2 完整复盘，2026-06-25）

### 规则文件
- [PPT 模板使用规范](ppt-master/ppt-template-rules.md) — 模板继承铁律 + image5全幅 + **Template-fill直通路径(v2.10.0)** + **Beautify工作流(v2.11.0)** + 多Deck摄入
- [PPT SVG 技术硬约束](ppt-master/ppt-svg-hard-rules.md) — **11 条导出必崩禁区**（新增规则 8-11：HTML实体、内部defs引用、XML属性空格、元素重叠；v2.11.0 + 融通复盘）
- [PPT 排版设计约束](ppt-master/ppt-layout-rules.md) — **规则0: Mode+Visual-Style双轴体系(v2.10.0+)** + 8 条视觉质量规范 + 18 种 Visual Style + 布局模式库(A-F) + Logo 强制规则 + **架构层一致性(6f)**

### 检查清单与方法论
- [PPT 搜图必须用 image_search.py 而非 WebSearch](ppt-master/ppt-image-search-must-use-image-search-py.md) — **Pexels Key 已配，严禁用 WebSearch 替代**（v2.10.0+ 图片生成后端已切换为 OpenAI 兼容接口）
- [PPT 生成前置检查清单](ppt-master/ppt-preflight-checklist.md) — 四阶段 46 条验证项（v2.11.0 + 融通复盘：新增 S7-S10 SVG技术项、L16 架构一致性、L17 来源行、T8 串行/并行选择、C14 批量修改安全）
- [PPT 并行生成铁律](ppt-master/ppt-parallel-generation-iron-law.md) — 6-8x 加速（Agent prompt 新增 Mode+Visual Style 传递）
- [PPT 并行/串行选择规则](ppt-master/ppt-v2110-parallel-still-valid.md) — spec_lock 含精确坐标时并行仍有效，**新增规则 7：生成前必须让用户选择串行/并行**（2026-06-25）
- [PPT 图表蓝本派生 vs 模板对比](ppt-master/ppt-chart-blueprint-vs-template.md) — 选用决策树（v2.7.0+ charts_index.json 已重构为结构命名）
- [PPT 图表选型矩阵](ppt-master/ppt-chart-selection-matrix.md) — 5 层结构提取规范 + v2.10.0+ 图表页内容质量
- [PPT 图表蓝本派生方法论](ppt-master/ppt-blueprint-derived-method.md) — HTML→SVG 三步法（定比例→算坐标→写 SVG），鱼骨图和汇入对比图两套布局参数
- [PPT SVG 高保真技法](ppt-master/ppt-svg-hifi-technique.md)

### 提示词模板
- [PPT 产品演示模板](ppt-master/prompt-template-ppt-product-demo.md)（v2.11.0 更新：新增 mode + visual_style 变量）

### 布局参照 SVG（生成前先读，直接套用几何结构）
- [`layout-ref-architecture-layered.svg`](ppt-master/layout-ref-architecture-layered.svg) — 四层架构图（changan P06）：数字编号 + 虚线边框 + 32px 色块卡片
- [`layout-ref-card-layered-with-number.svg`](ppt-master/layout-ref-card-layered-with-number.svg) — 编号卡片（财会 P06）：44px 编号方块 + 6px 左色条 + 【标签】前缀 + 底部结论条

## 自建技能规范
- [自建技能custom-前缀命名规范](custom-skill-prefix-convention.md) — 所有自建skill必须以custom-为前缀
- [PPT自定义图表custom-前缀规范](ppt-master/ppt-master-custom-chart-prefix.md) — 用户扩展的图表模板必须以custom-为前缀（文件名和charts_index.json key均适用）
- [投标文档模板继承铁律](ppt-master/bidding-doc-template-inheritance.md) — custom-doc-generator 必须用 Document(template_path)
- [PPT封面只生成一版](ppt-master/ppt-cover-single-version-no-web-bg.md) — 封面单版本，用模板默认背景，不搜网络图片做第二版

## 重要注意事项
- npm安装需用临时缓存: npm install --cache /tmp/npm-cache-xxx（~/.npm有权限问题）
- Python venv路径: backend/.venv/bin/python
- Playwright首次安装: .venv/bin/playwright install chromium

## v2.11.0 流程教训
- [v2.11.0 严禁跳过 Step4 八项确认](ppt-master/ppt-v2110-confirm-step-must-not-skip.md) — 升级后跳过确认直接生成，导致模板偏离(TOC坐标)+图表密度不足+质量下降；结构页必须照搬模板坐标
- [PPT 反模式大全](ppt-master/ppt-anti-patterns.md) — 融通项目 v1→v2 全量复盘，覆盖 17 个已知错误模式（模板漂移、HTML实体、sed破图标、元素重叠、架构层不一致、来源行自动添加等），生成前必读
- [PPT 并行/串行选择规则](ppt-master/ppt-v2110-parallel-still-valid.md) — spec_lock 含精确坐标时并行仍有效，**规则 7：生成前必须让用户选择串行/并行**（2026-06-25）
- [全幅白色rect违规记录 2026-06-25](ppt-master/ppt-full-white-rect-violation-20260625.md) — 融通项目 21 页全量违反规则1，根因是生成前未读记忆文件，模板中白色rect不能照抄
