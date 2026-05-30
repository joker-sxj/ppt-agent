---
name: ppt-agent
description: >
  端到端 PPT 生成助手:把"人类顶级 PPT 团队"的工作流——需求调研→大纲→资料检索→策划稿→整页
  SVG 设计——固化成流水线,产出可拖进 PowerPoint 编辑的 1280×720 整页 SVG 幻灯片 + 网页预览。
  当用户想做/生成/制作一套 PPT、幻灯片、演示文稿、slides、deck,或要为汇报/答辩/路演/组会/
  课堂/产品介绍准备演示,或说"用 ppt-agent"、"帮我做个关于 X 的 PPT"、"把这个主题做成幻灯片"
  时触发。全程两道人审关口(需求、大纲),其余自动。除整页 SVG + 网页预览外,确认后还自动
  产出可直接打开的 .pptx——**每个色块/文字/线条都是原生 PowerPoint 形状,打开即可改字改色**;
  若起点是已有 .pptx 模板、或要做幻灯片级 OOXML 增删 / 文本提取,请改用 pptx 技能。
---

# PPT Agent

## Overview
把"人类顶级 PPT 团队"的工作流固化成流水线:需求调研 → 大纲 → 资料检索 → 策划稿 → 整页 SVG 设计 → 预览交付。逐页产出 1280×720 的 SVG,再逐元素翻译成由原生形状 / 文本框组成、打开即可改字改色的 .pptx,并生成网页预览。核心信条:**PPT 的灵魂是内容不是皮囊**——先想清楚为谁做、做什么,再谈设计。

## 工作流总览
六阶段顺序执行;**只在 ① 需求、② 大纲两处停下等用户确认**,其余自动跑完。

| # | 阶段 | 关口 | 读取的提示词 / 资源 | 落盘 |
|---|---|---|---|---|
| 1 | 需求调研 | 🛑 等确认 | `prompts/01-requirement-interview.md` | `00-research.md` |
| 2 | 大纲 | 🛑 等确认 | `prompts/02-outline-architect.md` | `01-outline.json` |
| 3 | 资料检索 | 自动 | (工具驱动,见下) | `02-content.md` |
| 4 | 策划稿 | 自动 | `prompts/03-planning.md` + `references/bento-grid.md` | `03-plan.md` |
| 5 | SVG 设计 | 自动 | `prompts/04-design-svg.md` + `references/bento-grid.md` | `slides/slide-NN.svg` |
| 6 | 预览 + 出片 | 自动 | `scripts/build_preview.py`、`scripts/build_pptx.py`、`references/svg-to-powerpoint.md` | `preview.html`、`<主题>.pptx`、`README.md` |

## 开始前
确认或推断 PPT 主题,在当前工作目录下建产物目录 `ppt-decks/<今天日期>-<主题slug>/`(内含 `slides/` 子目录)。用户指定了别的位置就用用户的。

## 分阶段执行

### 阶段 1 · 需求调研 🛑
读 `prompts/01-requirement-interview.md` 照其执行:先联网调研主题,再向用户提 3-5 个关键问题;把调研摘要 + 需求纪要写入 `00-research.md`。**停下**,等用户补全 / 确认需求再继续。

### 阶段 2 · 大纲 🛑
读 `prompts/02-outline-architect.md`,把 `00-research.md` 填入 `{{CONTEXT}}`、目标页数填入 `{{PAGE_REQUIREMENTS}}`,生成 `[PPT_OUTLINE]` JSON 并提取存为 `01-outline.json`。再以"数字便利贴"形式**每页一行**展示给用户(封面 / 目录 / 各章节页 / 结尾),供其增删、改写、调序。**停下**,等用户确认大纲再继续。

### 阶段 3 · 资料检索(自动)
按确认后的大纲**逐页 / 逐节**用 WebSearch / Tavily 检索,为每页备齐要点、数据、案例(标注来源),汇总写入 `02-content.md`、按大纲结构组织。(用户已给足素材或主题无需外部资料时,可精简此步。)

### 阶段 4 · 策划稿(自动)
读 `prompts/03-planning.md` 与 `references/bento-grid.md`,结合 `01-outline.json` + `02-content.md`,为每页规划版面(便当组合、视觉层级、占位元素),写入 `03-plan.md`。

### 阶段 5 · SVG 设计(自动)
读 `prompts/04-design-svg.md` 与 `references/bento-grid.md`。**逐页**把该页在 `03-plan.md` 的规划填入 `{{PAGE_PLAN}}`、该页内容填入 `{{PAGE_CONTENT}}`、需求中的风格填入 `{{STYLE}}`,生成纯 SVG,存为 `slides/slide-01.svg`、`slide-02.svg`……(两位数序号,与页序一致)。

### 阶段 6 · 预览 + 出片(自动)
1. **预览**:运行 `scripts/build_preview.py <产物目录>`,生成 `preview.html`。
2. **出 .pptx**:运行 `scripts/build_pptx.py <产物目录> [<输出名>.pptx]`——把每页 SVG **逐元素翻译成原生 PowerPoint 形状**(矩形/圆角矩形、椭圆、连接线、文本框、渐变填充逐一还原),**打开即可直接改字、改色、挪位置**;只有复杂图标/箭头/装饰路径(`<path>` 与浅色 `<g opacity>`)合成一张透明 PNG 叠在最上层。依赖 `python-pptx`、`lxml`;图标叠层需任一 SVG 渲染器(Chrome/Edge / rsvg-convert / inkscape / cairosvg / LibreOffice,脚本自动探测,缺了就跳过图标);Windows 加 `PYTHONUTF8=1` 跑。
3. **README**:按 `references/svg-to-powerpoint.md` 写 `README.md`(主题 / 页数 / 风格 + 如何打开 .pptx,附手工导入 SVG 的兜底法)。

最后报告产物目录,提示用户直接打开 `<主题>.pptx`,或先看 `preview.html` 通览。

## 关键约束
- **不跳过两道关口**:需求、大纲未经用户确认,不得进入后续自动阶段。
- **SVG 严格 `viewBox="0 0 1280 720"`**:文字用 `<text>`(可编辑)、不引外部资源。
- **重要 PPT**:用户要求时可在阶段 4 后加设一道"看策划稿再出图"的确认关;默认不停。
- 阶段 6 自动产出 .pptx(每页逐元素翻译成原生形状 / 文本框,打开即可改字改色;仅复杂图标走透明 PNG 叠层)。但**若起点是已有 .pptx 模板、或要做幻灯片级 OOXML 增删 / 文本提取**,那是 `pptx` 技能的活,转过去。
