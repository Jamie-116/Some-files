---
name: body-style-always-indented
description: "Word文档正文统一用\"投标正文\"样式（首行缩进），禁止使用\"投标正文无首缩\""
metadata: 
  node_type: memory
  type: feedback
  originSessionId: dc6ae6fa-efe1-4def-bef1-7e6fc91b90b5
---

所有生成 Word 文档的正文段落必须使用"投标正文"样式（首行缩进 2 字符），禁止使用"投标正文无首缩"。

**已修改**：`custom-doc-generator/scripts/generate.py` 中 `body_first` 映射从 style ID "24" 改为 "23"，与 `body` 统一。

**Why:** 用户明确要求正文统一缩进，不缩进的段落不符合投标文档格式规范。

**How to apply:** 生成 content.json 时可以继续使用 `body` 和 `body_first` 区分段落逻辑（首段/后续段），但渲染到 Word 时两者都输出为"投标正文"样式。不要手动在 text 中写编号。
