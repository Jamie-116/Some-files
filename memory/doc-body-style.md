---
name: doc-body-style
description: "Word文档正文样式规范 — 全部使用\"投标正文\"（首行缩进2字符），禁止混用首缩/无缩进"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ad9e426e-af6f-4d26-bf96-01b3c3f1299f
---

使用 custom-doc-generator 生成 Word 文档时，所有正文段落统一使用 `"type": "body"`（对应模板样式"投标正文"，id=23，首行缩进 2 字符）。

**禁止使用 `body_first`（投标正文无首缩，id=24，无缩进）**。即使章节首段也不能用 `body_first`——全部段落一律 `body`。

**Why**: 用户明确偏好全文档正文都有首行缩进，不需要"每个章节第一段无缩进"这个差别。
**How to apply**: 写 content.json 时，所有正文段落 type 只用 `"body"`，不使用 `"body_first"`。
