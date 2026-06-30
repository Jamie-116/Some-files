---
name: ppt-cover-single-version-no-web-bg
description: 封面只生成一版，使用模板默认背景，不搜索网络图片做第二版封面
metadata: 
  node_type: memory
  type: feedback
  priority: high
  date: 2026-06-25
  originSessionId: d8771460-c0ae-4593-9d6b-9d4643edd271
---

# 封面只生成一版，不用网络图片背景

**规则**：封面只生成一个版本，直接使用模板默认背景（`01_cover.svg` 中指定的 `image4.jpeg` 等），**不要**用 `image_search.py` 搜索网络图片生成第二版封面。

**Why**：用户明确要求"封面页不需要生成两个了，不要生成带图片的封面"。网络搜图的第二版封面增加不必要的复杂度和文件大小，模板自带背景已足够。

**How to apply**：
- `design_spec.md §IX` 中只规划一个封面页（P01）
- 封面直接继承模板 `01_cover.svg` 的背景图片，不额外搜图
- `spec_lock.md images` 中不需要 `cover_bg_alt.jpg` 等备选封面图片
- Step 5 图片采集阶段不为封面搜图
