---
name: daily-news-brief-cn
description: 当用户要求获取中文新闻摘要、每日简报，或要求将新闻整理成报纸风长图/简报图时使用。该技能会先从可信中文 RSS 源抓取并汇总新闻，再将结果整理为适合手机阅读的报纸风 HTML，并可进一步截图导出为 PNG。最终产物统一输出到 media/daily-news-brief-cn/ 下。
---

# Daily News Brief CN

## Overview

这是一个合并技能，整合了：

1. **新闻抓取与摘要**：从中文 RSS 源抓取新闻并生成结构化摘要
2. **报纸风渲染**：将摘要渲染为适合手机阅读的报纸风长图

目标是形成一条完整流水线：

```bash
RSS feeds -> headlines & summaries -> structured JSON -> HTML -> PNG
```

---

## 适用场景

当用户提出以下类型请求时，应优先使用本技能：

- 中国新闻摘要
- 科技新闻汇总
- 商业新闻汇总
- 每日新闻简报

也适用于组合型请求，例如：

- “给我生成今天的中文科技新闻简报，并做成报纸图”
- “抓一下今天中国和科技新闻，整理成适合飞书发送的长图”

---

## 数据源

### People.cn（主流国内新闻）

```bash
# Politics
curl -s "http://www.people.com.cn/rss/politics.xml"

# Society
curl -s "http://www.people.com.cn/rss/society.xml"
```

### 36Kr（科技）

```bash
curl -s "https://www.36kr.com/feed"
```

### Huxiu（商业/评论）

```bash
curl -s "https://rss.huxiu.com/"
```

---

## RSS 解析方式

提取标题与描述：

```bash
curl -s "https://www.36kr.com/feed" | \
  grep -E "<title>|<description>" | \
  sed 's/<[^>]*>//g' | \
  sed 's/^[ \t]*//' | \
  head -30
```

---

## 默认工作流

1. 抓取 People.cn 新闻
2. 视需要补充 36Kr 和 Huxiu
3. 提炼 5-8 条重点新闻
4. 按主题分组（政治 / 社会 / 科技 / 商业）
5. 生成结构化简报 JSON
6. 将 JSON 渲染为报纸风 HTML
7. 最后，进一步截图导出为 PNG

如果用户已经给出了新闻摘要、要点列表或 JSON，可跳过抓取和摘要步骤，直接从结构化内容渲染为 HTML/PNG。

---

## 输出目录与文件名要求

所有生成产物默认输出到 OpenClaw 工作区：

```bash
media/daily-news-brief-cn/
```

其中最终图片文件必须保存为：

```bash
media/daily-news-brief-cn/daily-news-brief-cn.png
```

要求：

- 若目录不存在，先创建目录
- 若目录下已有文件，先清空
- HTML、normalized JSON 等中间产物也建议一并保存在同目录下，便于调试

推荐输出文件：

```bash
media/daily-news-brief-cn/daily-news-brief-cn.json
media/daily-news-brief-cn/daily-news-brief-cn.normalized.json
media/daily-news-brief-cn/daily-news-brief-cn.html
media/daily-news-brief-cn/daily-news-brief-cn.png
```

---

## 结构化输出格式（推荐中间层）

在渲染前，优先整理成如下 JSON：

```json
{
  "paper_name": "今日简报",
  "issue": "2026-03-17",
  "title": "中国与科技要闻速览",
  "subtitle": "每日国内、科技与商业重点新闻摘要",
  "summary": "一句导语，概述今天最值得关注的动态。",
  "highlights": [
    "3-5 条最重要的 TL;DR",
    "每条尽量短句",
    "适合侧栏速览"
  ],
  "quote": "可选，放一句高度概括的判断。",
  "sections": [
    {
      "heading": "国内",
      "bullets": ["新闻1", "新闻2"]
    },
    {
      "heading": "科技",
      "bullets": ["新闻1", "新闻2"]
    },
    {
      "heading": "商业",
      "bullets": ["新闻1", "新闻2"]
    }
  ],
  "footer_note": "来源：人民网 / 36Kr / 虎嗅｜整理：AI"
}
```

---

## 标题生成规则

- 标题最多两行
- 不使用 `...` / `……` 截断
- 优先重写成更短、更像报纸标题的表达
- 如仍过长，再轻微缩小字号
- 不要直接把原始长句硬塞进标题区

---

## 报纸风版式

v1 固定为手机版长图：

- 报头（刊名 / 期号 / 日期）
- 大标题
- 导语
- 左侧主内容区（sections）
- 右侧速览栏（highlights + quote）
- 页脚（来源 / 整理说明）

风格：

- 黑白报纸
- 轻米色纸张背景
- 少量强调线
- 优先适合手机端阅读与飞书查看

---

## 渲染脚本

### 1) 先确保输出目录存在

```bash
mkdir -p media/daily-news-brief-cn
```

### 2) 导出 HTML

```bash
python skills/daily-news-brief-cn/scripts/render_newspaper.py \
  --input media/daily-news-brief-cn/daily-news-brief-cn.json \
  --html media/daily-news-brief-cn/daily-news-brief-cn.html \
  --normalized media/daily-news-brief-cn/daily-news-brief-cn.normalized.json
```

### 3) 导出 PNG（覆盖旧文件）

```bash
python skills/daily-news-brief-cn/scripts/render_newspaper.py \
  --input media/daily-news-brief-cn/daily-news-brief-cn.json \
  --html media/daily-news-brief-cn/daily-news-brief-cn.html \
  --png media/daily-news-brief-cn/daily-news-brief-cn.png
```

脚本应尝试调用本机 Chromium 进行 headless screenshot。若 PNG 导出失败，HTML 仍应保留并作为可用结果。若目标 PNG 已存在，应直接覆盖旧文件，不需要保留历史版本。

---

## 质量检查

完成前至少检查：

- 新闻是否来自可信源
- 是否控制在 5-8 条重点
- 分类是否清晰
- 标题是否像报纸标题，而非聊天句子
- highlights 是否够短
- sections 是否控制在 2-4 个
- 手机上是否能一眼抓住重点
- 若 PNG 失败，HTML 是否成功生成

---

## 推荐执行策略

优先使用以下顺序：

1. 抓新闻
2. 去重与筛选
3. 提炼结构化要点
4. 生成 JSON
5. 渲染 HTML
6. 可用时截图为 PNG

不要把未经整理的大段原文直接塞进模板，除非用户明确要求保留原文风格。

---

## 最佳实践

- 保持摘要简洁，优先 5-8 条
- 兼顾主流新闻与科技/商业媒体
- 使用结构化 JSON 作为中间层，提升稳定性与复用性
- 让渲染模块只负责展示，不再承担内容理解任务

---

## 一句话理解

这个合并技能负责：

**从中文新闻源自动生成新闻摘要，并将其渲染为可直接发送的报纸风简报图。**
