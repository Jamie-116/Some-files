---
name: ppt-image-search-must-use-image-search-py
description: PPT 搜图必须用 image_search.py 而非 WebSearch — WebSearch 只返回文本链接不下图片（v2.11.0 更新）
metadata: 
  node_type: memory
  type: feedback
  priority: highest
  originSessionId: 4867e3a2-4475-403e-b8f9-abd07f41b8c4
  updatedForVersion: "v2.11.0"
  lastUpdated: "2026-06-24"
---

2026-06-12 陕西德联项目。P02 封面搜索背景图时，错误地使用了 `WebSearch` 工具替代 `image_search.py`，导致只拿到 iStock/Getty 付费链接，没有可下载图片。

**v2.10.0+ 变更**：图片生成后端已切换为 **OpenAI 兼容接口**（替代各供应商独立后端）。`image_search.py` 仍为搜图入口，但 AI 生成图片路径有变化。检查 `~/.ppt-master/.env` 中是否有新增的图片生成 API 配置。

## 核心教训

**WebSearch ≠ 图片搜索**。`WebSearch` 是文本搜索工具，返回的是网页标题和 URL 摘要，它不会、也不能下载图片文件。

## 正确做法

PPT Master 搜索配图必须使用 `scripts/image_search.py`：

```bash
python3 scripts/image_search.py "search keywords" \
  --filename cover_bg.jpg \
  --provider pexels \
  --orientation landscape \
  -o <project_path>/images/
```

## 图片搜索 API 配置

Pexels API Key 已配置在 `~/.ppt-master/.env`：
```
PEXELS_API_KEY=your_pexels_api_key_here
```

该 Key 免费（200次/小时），质量好，不需要 attribution。

## 可用 Provider

| Provider | 需要 Key | 质量 |
|----------|---------|------|
| `pexels` | 是 | 好，商业图库风格 |
| `pixabay` | 是 | 好，需单独注册 |
| `openverse` | 否（默认） | 不稳定 |
| `wikimedia` | 否 | 参差不齐 |

**Why**: WebSearch 返回的是文本链接而非图片文件，导致封面第二版只能用模板背景图 fallback。image_search.py 才是 PPT Master 管道中真正的图片获取入口，支持 Pexels/Pixabay/Openverse/Wikimedia 四个 provider，质量可控且能直接下载到项目的 images/ 目录。

**How to apply**: 每次 PPT 生成流程中，Step 5 Image Acquisition 阶段如果设计规格中有 `web` 类型的图片行，必须运行 `image_search.py`，不能跳过或改用 WebSearch 代替。即使只搜索 1 张图，也要跑这个脚本而不是"先 WebSearch 看看有没有"。
