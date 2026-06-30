---
name: bidding-doc-template-inheritance
description: custom-doc-generator 技能必须始终以 投标模板3.docx 为基底生成文档
metadata: 
  node_type: memory
  type: feedback
  originSessionId: cc6af0f5-578c-4b34-8676-087bccc6ecd7
---

使用 `custom-doc-generator` 技能时，**必须**以 `assets/投标模板3.docx` 为基底，即用 `Document(template_path)` 打开模板后再清空 body 写入内容，不可用 `Document()` 空文档。

**Why:** 模板内定义了完整的样式体系（Heading 1~8、投标正文、投标正文首缩）和页面设置（A4、上下 2.54cm、左右 3.17cm），`Document()` 空文档不包含这些，导致输出格式与投标规范不一致。

**How to apply:**
1. `doc = Document(template_path)` 打开模板
2. 保留 `sectPr`（页面设置），清空 body 其余内容
3. 用 `doc.styles["Heading 1"]` 等模板样式名写入标题和正文
4. 对于 `content.json` 不支持的表格/图片，用 python-docx API 手动扩展写入
5. 不可绕开模板直接用 `Document()` 生成

参见 [[ppt-template-inheritance-rule]] 类似教训。
