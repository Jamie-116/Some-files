---
name: prompt-template-ppt-product-demo
description: 固定提示词模板：B端政企产品演示PPT生成，变量为源文档路径（v2.11.0 更新，已适配 Mode+Visual Style 双轴体系）
metadata: 
  node_type: memory
  type: project
  originSessionId: f8f991c3-9762-4392-8200-bcae07a7c264
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

## 触发方式

用户在对话框中输入 `@prompts/ppt-product-demo.md` 或提及"产品演示PPT"模板。

## 变量

- `{{DOC_PATH}}` — 源文档路径（.docx 文件）
- 隐式变量：`mode`（默认 `pyramid`）、`visual_style`（默认 `swiss-minimal`），由 Step 4 交互式确认页锁定
- 品质增强级别：`Standard`（默认）/ `Premium` / `Ultimate`，用户输入切换

## 执行流程

1. 读取 `prompts/ppt-product-demo.md`
2. 如果 `{{DOC_PATH}}` 仍是占位符、未被替换：**追问用户**「请指定源文档路径（如 `@docs/xxx.docx`）：」
3. 用户提供路径后，将 `{{DOC_PATH}}` 替换为实际路径
4. v2.11.0 Step 4 启动交互式确认页（`confirm_ui/server.py --daemon --wait`），用户锁定 mode + visual_style + 色板 + 字体 + 图片策略等八项
5. 并行 Agent 生成 SVG 页面（spec_lock.md 含 mode + visual_style 字段，Agent prompt 必须嵌入全局风格参数）
6. 校验阶段执行 v2.11.0 版 preflight checklist（含 C10 Visual Style 一致性、C11 Mode 一致性、C12 封面结尾）

## v2.8.0 → v2.11.0 主要变更

| 旧提示词 | 新提示词 | 说明 |
|---------|---------|------|
| 设计强度 A/B/C/D | Mode + Visual Style + 品质增强级别 Standard/Premium/Ultimate | v2.10.0 架构级变更 |
| 无确认页 | Step 4 交互式可视化确认页 | v2.11.0 新增 |
| 封面两版规则 | 封面 hook + composition（v2.11.0 强制） | 不再接受通用标题页 |
| 无结尾约束 | 结尾 final-impression takeaway（v2.11.0 强制） | 不能是空洞"谢谢" |
| charts_index.json (71个) | charts_index.json（v2.7.0 结构命名重构） | 文件名描述视觉结构非领域模型 |
| 无 refine-spec | refine-spec 可选工作流 | v2.10.0 新增 |
| 无 template-fill/beautify | template-fill / beautify-pptx 可选工作流 | v2.10.0/v2.11.0 新增 |
| 无 LaTeX | LaTeX 公式渲染（4 供应商后备链） | v2.9.0 新增 |
| 图片生成多后端 | OpenAI 兼容接口 | v2.10.0 切换 |
| 无 mode/visual_style 传递 | Agent prompt 必须嵌入 mode + visual_style | 并行生成约束传递 |

## 关联

[[ppt-template-rules]]（含 Template-fill、Beautify）
[[ppt-svg-hard-rules]]（含 LaTeX 渲染规则6、手绘类规则7）
[[ppt-layout-rules]]（规则0: Mode + Visual Style 双轴体系，18 种 Visual Style）
[[ppt-preflight-checklist]]（v2.11.0 版，含 C10-C13）
[[ppt-memory-vs-skill-priority]]（潜在冲突 6-8）
[[ppt-parallel-generation-iron-law]]（Agent prompt 须含 mode+visual_style）
