---
name: custom-skill-prefix-convention
description: 自建技能必须以custom-为命名前缀
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a2e387a4-f128-47e6-82f8-0e0e644b4bf7
---

所有用户自建的 skill 必须以 `custom-` 作为名称前缀，以区分自建技能和从社区/第三方下载的技能。

**当前自建技能清单（均带 custom- 前缀）：**
- `custom-drawio-flowchart` — Draw.io 业务流程图
- `custom-effort-estimator` — 开发工作量评估
- `custom-python-docker-deploy` — Python Docker 部署
- `custom-voice-master` — 配音大师
- `custom-doc-generator` — 通用文档生成器（已从 custom-bidding-doc-generator / bidding-doc-generator 重命名）
- `custom-demo-spec-generator` — Demo 需求规格说明书生成器（已从 demo-spec-generator 重命名）
- `custom-chart-builder` — 为 ppt-master 扩展 SVG 图表模板（缺口识别→设计→编写→校验→注册→测试页转 PPTX）

**Why:** 在工作区 `.claude/skills/` 目录下存在自建和第三方下载的技能混放，没有前缀时无法直观区分来源。

**How to apply:** 新建技能时统一使用 `custom-<功能名>` 格式命名；发现未加前缀的自建技能时主动重命名（移动目录 + 修改 SKILL.md 的 name 字段）。
