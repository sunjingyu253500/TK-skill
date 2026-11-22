---
name: realistic-ugc-shot
description: 提供"真实感手持手机实拍 / TikTok 素人实拍 / 无 AI 痕迹"风格的 UGC 图片与视频提示词配方。本技能应在用户要求实拍感素材、门店随手拍、产品开箱、口播自拍、用户评价图或视频、自然皮肤质感与毛孔、9:16 竖屏、raw 无滤镜风格，或提供类似 "Real handheld phone shot ... --ar 9:16 --style raw" 的提示词并希望复用与改造时使用。
agent_created: true
---

# Realistic UGC Shot（真实感手持实拍）

## Overview

将"真实感手持手机拍摄"的视觉风格固化成可复用的提示词配方，用于在 WorkBuddy 中稳定产出 TikTok / 跨境电商场景的素人实拍感图片与视频。核心目标是可信度：画面要像真人用手机随手拍下的日常片段，而不是精修广告或 AI 渲染图。

## When to Use

- 生成"像真实用户拍的"产品图、开箱图、门店试用图、评价配图。
- 生成 TikTok / Reels / 短视频用的竖屏实拍感视频或视频首帧。
- 用户抱怨 AI 图"塑料脸、磨皮过重、滤镜感强、太完美、一眼假"并要求修正。
- 用户提供一段实拍风格英文提示词（含 `--ar 9:16 --style raw` 等参数）希望复用、改造、批量套用。
- 需要统一一组素材的调性（同一套光照、颗粒、肤色、构图语言）。

## Core Workflow

1. **判定产出类型**：静态图片 → 图像生成；动态视频 → 视频生成；要求"同一人物/同一场景连续镜头" → 先生成一张基准静帧，再以该帧作为参考图或首帧生成视频。
2. **读取配方**：加载 `references/style-recipe.md`，取用其中的基础风格串、分层模块、场景模板与负向词库。
3. **组装提示词**：按 `场景主体 → 动作 → 环境 → 风格配方 → 负向约束` 的顺序拼接。可用 `scripts/build_prompt.py` 快速生成，也可在对话中按同一结构手工拼接。
4. **调用生成工具**：先通过工具检索获取 `ImageGen`（图片）或 `VideoGen`（视频）的参数结构，再按结构传参。不确定参数时按工具返回的 schema 填写，不要臆造字段。
5. **质检**：逐项核对下方 Quality Checklist；任一项不达标则只调整对应模块，不要整体重写提示词。
6. **迭代**：一次只改一个变量（光线 / 镜头距离 / 皮肤质感 / 动作幅度），便于定位有效改动。

## Tool Routing

| 需求 | 工具 | 关键做法 |
| --- | --- | --- |
| 单张实拍感图片 | ImageGen | 提示词中保留 9:16 竖屏构图与 raw 无滤镜描述 |
| 实拍感短视频 | VideoGen | 强调轻微手持位移与自然微动作，避免电影级运镜 |
| 同一人物多镜头一致 | ImageGen → VideoGen | 先出基准静帧，再把该帧作为参考图或首帧 |
| 已有产品图转实拍感 | ImageGen（图生图） | 保留产品主体与结构，只替换光照、质感与背景氛围 |

说明：`ImageGen` 与 `VideoGen` 属于按需加载工具，需要先检索工具结构再调用。

## Prompt Assembly

基础风格串（原文，未经改动）：

```text
Real handheld phone shot, casual in-store shooting, natural indoor light, soft shadows, slightly unpolished real footage, no heavy filter, daily life atmosphere, natural human skin texture, even facial lighting, realistic pores, subtle camera movement, no distortion, no glitch, no plastic-looking skin, ultra realistic, natural color grading, TikTok real user video style, no obvious AI trace --ar 9:16 --style raw
```

组装规则：

- `--ar 9:16` 与 `--style raw` 是 Midjourney 风格参数。内置生成工具若不接受参数，改写为自然语言：`vertical 9:16 portrait framing, raw unstylized rendering, no heavy filter`，不要直接把参数当作文字传入。
- 需要负向提示词字段时，把 `no heavy filter / no distortion / no glitch / no plastic-looking skin / no obvious AI trace` 拆出作为负向词，正向串中保留其正向表达（clean natural rendering、natural skin texture）。
- 主体描述放在最前，风格串放在最后；不要插入"高清、8K、电影感、杰作"等会把画面推向精修广告的词。
- 详细模块库、场景模板与诊断表见 `references/style-recipe.md`。

## Quality Checklist

- [ ] 画面为竖屏 9:16，主体偏中上或按手机随手拍的略偏构图，不是居中对称的产品棚拍。
- [ ] 光源可解释：室内顶灯 / 窗光 / 货架灯之一，面部受光均匀，无阴阳脸与过曝。
- [ ] 皮肤保留毛孔与纹理，无磨皮蜡像感；手指、耳廓等细节结构正常。
- [ ] 轻微手持感：画面有很小幅度的晃动或轻微运动模糊，但没有抖动到看不清主体。
- [ ] 色彩接近手机直出：自然色温、不过度饱和、无重滤镜与夸张 HDR。
- [ ] 无文字乱码、无多余肢体、无结构变形（把任何 distortion / glitch 视为失败项）。
- [ ] 整体第一眼像生活记录，不像广告大片或 3D 渲染。

## Common Pitfalls

- 直接把 `--ar 9:16 --style raw` 原样塞给不支持参数的工具 → 改写为自然语言构图与风格描述。
- 加入 `beauty filter`、`smooth skin`、`cinematic lighting`、`studio shot` → 立刻产生塑料脸与广告感，与配方冲突。
- 把"手持"写成 `shaky camera` → 容易生成过度抖动；用 `subtle camera movement` 或 `slight handheld drift`。
- 为了"真实"而过度加颗粒或暗角 → 变成刻意做旧，反而暴露 AI 痕迹。
- 一次修改多个变量导致无法判断哪个词有效 → 每轮只改一个模块。

## Resources

- `references/style-recipe.md`：原始提示词逐字版、分层模块库、场景模板、负向词库、问题诊断表。
- `scripts/build_prompt.py`：按模块组装并输出最终提示词的小工具。
