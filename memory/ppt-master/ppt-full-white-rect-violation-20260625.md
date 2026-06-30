---
name: ppt-full-white-rect-violation-20260625
description: 融通项目全幅白色背景rect违规记录 — 21页全部违反规则1，生成前未读记忆文件
metadata: 
  node_type: memory
  type: feedback
  priority: highest
  date: 2026-06-25
  project: rongtong-ai-platform
  originSessionId: d8771460-c0ae-4593-9d6b-9d4643edd271
---

# 全幅白色背景 rect 违规事件

**日期**：2026-06-25
**项目**：融通智算底座与大模型服务中台建设方案（21 页 PPT）

## 问题

生成的全部 21 个 SVG 文件均包含 `<rect x="0" y="0" width="1280" height="720" fill="#FFFFFF"/>`，违反 [[ppt-svg-hard-rules]] 规则 1。

## 根因

生成前未读取记忆文件中的 PPT SVG 技术硬约束。Executor 阶段跳过了记忆文件读取步骤，直接参考模板 SVG（模板中也有此 `<rect>`，属于模板源文件的已知问题——Edge case："模板 SVG 中即使有白色背景 `<rect>`，不要照抄"）。

## 修复

`sed` 批量删除所有文件中的全幅白色 `<rect>` 行，重新 finalize + export。

## 如何避免

**每次 PPT 工作流启动时，必须按以下顺序读取**：
1. `[[ppt-memory-vs-skill-priority]]` — 优先级铁律
2. `[[ppt-svg-hard-rules]]` — 11 条 SVG 禁区
3. `[[ppt-anti-patterns]]` — 17 个已知错误模式
4. `[[ppt-template-rules]]` — 模板继承铁律

**Why**：记忆规则是实际导出踩坑后的硬教训，优先级高于 SKILL.md 和模板默认值。模板中的白色背景 `<rect>` 不能被照抄。

**How to apply**：生成第一个 SVG 之前，强制检查"当前页是否有全幅纯色背景 rect？白色→不写，图片→只写 `<image>`"。每写完一页 SVG，立即自查规则 1-11。
