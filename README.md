# 📰 Daily News Brief CN

An OpenClaw skill that generates daily Chinese news briefs from RSS feeds and renders them into mobile-friendly newspaper-style images.

---

## ✨ Features

- Fetch news from Chinese RSS sources (People.cn, 36Kr, Huxiu)
- Summarize and structure key stories
- Render into newspaper-style HTML → PNG
- Optimized for mobile (Feishu / chat sharing)
- Fully automated pipeline

---

## 🚀 Usage

### In OpenClaw

Place this skill in your OpenClaw skills directory and invoke with:

- “生成今日新闻简报”
- “整理中国和科技新闻并做成报纸图”
- “生成一份飞书可用的新闻简报”

The skill will automatically:

1. Fetch RSS news  
2. Summarize and structure content  
3. Save outputs to:

```bash
media/daily-news-brief-cn/
```

Final image:

```bash
media/daily-news-brief-cn/daily-news-brief-cn.png
```

---

### Manual Rendering (Optional)

If you already have structured JSON:

```bash
mkdir -p media/daily-news-brief-cn

python skills/daily-news-brief-cn/scripts/render_newspaper.py \
  --input media/daily-news-brief-cn/daily-news-brief-cn.json \
  --html media/daily-news-brief-cn/daily-news-brief-cn.html \
  --normalized media/daily-news-brief-cn/daily-news-brief-cn.normalized.json \
  --png media/daily-news-brief-cn/daily-news-brief-cn.png
```

---

## 🧩 Workflow

```text
RSS → Summary → Structured JSON → HTML → PNG
```

---

## 🙏 Acknowledgements

Inspired by:

- news-summary  
  https://clawhub.ai/joargp/news-summary  

- newspaper-brief  
  https://github.com/EisonMe/newspaper-brief  

---

## 📄 License

MIT License

This means you are free to use, modify, distribute, and even use this project commercially, as long as you include the original license.

---

## 🧠 Philosophy

> Turn information into product.
