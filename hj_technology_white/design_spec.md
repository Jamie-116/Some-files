---
deck_id: hj_technology_white
kind: deck
summary: 浩鲸科技中文模板（白）— 科技蓝白配色，左侧边栏章节页，全幅背景内容页
canvas_format: ppt169
page_count: 6
primary_color: "#0082FF"
page_types: [cover, toc, chapter, content, ending]
keywords: [科技, 企业汇报, 蓝白, 简洁, 浩鲸]
category: general
tone: Modern, professional, tech-forward
applicable_scenarios:
  - 企业汇报
  - 产品发布
  - 技术方案
  - 项目答辩
theme_mode: light
fonts:
  title: "FZLanTingHeiS-M-GB, Microsoft YaHei, PingFang SC, sans-serif"
  body: "FZLanTingHeiS-R-GB, Microsoft YaHei, PingFang SC, sans-serif"
  latin: "Roboto, Arial, sans-serif"
placeholders:
  TITLE: "封面标题"
  SUBTITLE: "副标题"
  DATE: "日期"
  TOC_TITLE: "目录标题"
  TOC_SUBTITLE: "目录副标题"
  CHAPTER_01_NUM: "章节编号"
  CHAPTER_01_TITLE: "章节标题"
  CHAPTER_NUMBER: "章节数字"
  CHAPTER_LABEL: "章节标签"
  CHAPTER_TITLE: "章节标题"
  CHAPTER_SUBTITLE: "章节副标题"
  PAGE_TITLE: "页面标题"
  CONTENT_AREA: "正文内容区域"
  COLUMN_LEFT: "左栏内容"
  COLUMN_RIGHT: "右栏内容"
  ENDING_TITLE: "结束语"
  ENDING_SUBTITLE: "结束语英文"
---

# 浩鲸科技中文模板（白）— Design Spec

## I. Template Overview

品牌企业 PPT 模板，以白色为基底，搭配科技蓝（`#0082FF`）、青色（`#28E6E6`）、紫色（`#8254F0`）为强调色。左侧边栏装饰用于章节和目录页，内容页采用全幅背景。

## II. Color Scheme

| Role | HEX | Purpose |
| ---- | --- | ------- |
| **Background** | `#FFFFFF` | 页面背景 |
| **Primary** | `#0082FF` | 核心主色，强调元素 |
| **Accent Cyan** | `#28E6E6` | 日期边框、装饰元素 |
| **Accent Purple** | `#8254F0` | 标题下划线 |
| **Accent Magenta** | `#C94CEB` | 封面分隔线 |
| **Dark Text** | `#2E2E32` | 主文字 |
| **Secondary Text** | `#595757` | 副文字 |

## III. Typography

- **Title**: 方正兰亭中粗黑简体 → `Microsoft YaHei`
- **Body**: 方正兰亭准黑简体 → `Microsoft YaHei`
- **Latin**: Roboto → `Arial`
- **PPT-safe fallback**: `Microsoft YaHei` for all CJK roles

## IV. Signature Design Elements

- **左侧边栏**: 424px 宽装饰图（`image5.png`），用于章节和目录页
- **Logo**: 右上角品牌标志（`image6.png` 内容页 / `image3.png` 封面结尾页）
- **封面分隔线**: 紫色 `#C94CEB` 装饰线
- **日期标签**: 青色 `#28E6E6` 圆角边框
- **全幅背景**: 内容页使用全幅背景图

## V. Page Roster

| # | File | Type | Description |
|---|------|------|-------------|
| 01 | `01_cover.svg` | Cover | 全幅背景 + 居中标题 + 副标题 + 日期（青色圆角边框） + 紫色分隔线 + Logo |
| 02 | `02_toc.svg` | TOC | 左侧边栏 + 目录标题 + 章节列表（编号 + 标题） + Logo |
| 03 | `02a_chapter.svg` | Chapter | 左侧边栏 + 章节编号（大字） + 章节标签 + 右侧章节标题 + 副标题 + Logo |
| 04 | `03a_content_full.svg` | Content (Single Column) | 全幅背景 + 页面标题 + 正文区域 + Logo |
| 05 | `03b_content_two_col.svg` | Content (Two Column) | 全幅背景 + 页面标题 + 左右双栏内容 + Logo |
| 06 | `04_ending.svg` | Ending | 全幅背景 + 居中感谢语 + 英文副标题 + Logo |

## VI. Template Assets

| Asset | Usage |
|-------|-------|
| `image3.png` | Logo（封面、结尾页） |
| `image4.jpeg` | 封面全幅背景 |
| `image5.png` | 左侧边栏装饰（目录、章节页） |
| `image6.png` | Logo（内容、目录、章节页） |
| `image7.jpeg` | 单栏内容页全幅背景 |
| `image8.jpeg` | 双栏内容页全幅背景 |
| `image9.jpeg` | 结尾页全幅背景 |
