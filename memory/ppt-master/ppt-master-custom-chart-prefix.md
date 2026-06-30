---
name: ppt-master-custom-chart-prefix
description: 用户自定义图表模板必须以 custom- 为前缀命名
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 2cd493af-d033-48ed-84b7-c3d61ceb206a
---

所有用户在 `templates/charts/` 下新增的 SVG 图表模板，文件名必须以 `custom-` 为前缀（如 `custom-decision_tree.svg`），在 `charts_index.json` 中的 key 也同步使用 `custom-` 前缀。

**Why:** 用户要求以此区分"技能自带的图表"和"用户自行扩展的图表"，与自建技能的 `custom-` 前缀命名规范保持一致。

**How to apply:** 新增图表时：
1. SVG 文件名：`custom-<name>.svg`
2. charts_index.json key：`"custom-<name>"`
3. `meta.total` 同步 +1
