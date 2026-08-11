---
name: social-media-news-cards
description: "将组装好的 HTML 卡片导出为高清 JPG 图片。输入 JSON/Markdown 格式的卡片数据，输出到按周期命名的目录。触发词：导出卡片、生成周报图片、卡片转图片、JPG 导出、weekly card export。"
version: 2.0.0
author: IGEBio-Synapse
license: MIT
metadata:
  hermes:
    tags: [cards, export, jpg, xiaohongshu, weekly-report]
    related_skills: [hermes-intel]
---

# 新闻卡片 HTML→JPG 导出器

## 概述

将已组装好的 HTML 卡片页面导出为独立 JPG 图片文件。
仅做导出，不做新闻数据搜集（那是另一个 SKILL 的工作）。

## 触发条件

- 用户说"导出卡片"、"生成周报图片"、"卡片转图片"、"JPG 导出"
- HTML 已就绪，需要导出为图片用于自媒体发布

## 输入格式

接收 JSON 或 Markdown，包含以下字段：

`json
{
  "title": "每周新闻快报",
  "year": 2026,
  "week": 32,
  "date_range": "08.03 — 08.10",
  "intro": "本周导读文案...",
  "highlights": ["新闻1摘要", "新闻2摘要", "新闻3摘要"],
  "cards": [
    {
      "num": 1,
      "category": "技术前沿",
      "date": "08-07",
      "title": "新闻标题",
      "summary": "摘要文案",
      "image": "imgs/01.jpg",
      "tags": ["标签1", "标签2"],
      "deep": [
        {"label": "原理", "text": "解读内容"},
        {"label": "意义", "text": "解读内容"}
      ],
      "source": "来源媒体",
      "maturity": "实验室"
    }
  ]
}
`

或 Markdown 格式（知识库 export 输出亦可）。

## 固定规范（不可改）

### 卡片布局

| 部分 | 内容 |
|---|---|
| 封面 | 标题 + 周期 + 导读 + 3 条高亮 |
| 新闻卡片 | 编号/图片/标题/摘要/标签/深度解读/底部来源 |
| 末页 | 完 |

### 视觉规格

| 参数 | 值 |
|---|---|
| 宽度 | 400px |
| 高度 | 600px（最小，内容溢出则自适应） |
| 圆角 | **0px（直角）** |
| 分辨率 | **3x DPI（Retina）** |
| 截图质量 | 95% |
| 图片格式 | JPEG（FF D8 头验证） |

### 字号层级（手机优先）

| 元素 | 字号 |
|---|---|
| 封面大标题 | 34-36px |
| 新闻标题 | 22px |
| 新闻摘要 | 14.5-15px |
| 深度解读 | 13.5px |
| 底部来源 | **12px**（放大，手机可读） |
| 标签 | 11px |

### 封面设计

- 标题：朴素直接（如"每周新闻快报"）
- 周期：标题下方写"2026年第32周（08.03 — 08.10）"
- 导读：封面中部开始写 editorial 导语
- 高亮：底部 3 条重点新闻（编号 + 一行文字）

### 禁忌

- ❌ emoji
- ❌ SVG 占位图
- ❌ Unsplash 等通用图库
- ❌ "下期预告"
- ❌ 圆角
- ❌ 深色模式（默认浅色）

## 可变参数（每次可能不同）

| 参数 | 说明 | 示例 |
|---|---|---|
| 封面配色 | 渐变起始/结束色 | #D4A574 → #A16207 |
| 标题文字 | 每期可能不同 | "每周新闻快报" |
| 导读文案 | 每周不同 | "本期关键发现..." |
| 高亮条目 | 3 条，每周不同 | ["新闻1", "新闻2", "新闻3"] |

## 图片来源策略

### 优先级

1. **本地文件**：imgs/ 目录已有图片
2. **Tavily 搜索**：精确搜索文章标题，include_images: true
3. **来源页提取**：访问文章 URL 提取 og:image

### 不接受

- Unsplash、Pexels 等通用图库（图文无关）
- 任何无法验证为新闻原文的图片来源

### 图片验证

`python
# 检查 magic bytes
data = open(path, 'rb').read()
assert data[:2] == b'\xff\xd8'  # JPEG
assert len(data) > 1000         # 排除验证码
`

### 格式统一

所有图片转换为 JPG：Image.save('JPEG', quality=90)

## 输出目录结构

`
<项目根目录>/
└── {标题}/
    ├── {YYYY}年第{NN}周/
    │   ├── card_01.jpg
    │   ├── card_02.jpg
    │   └── ...
    └── {YYYY}年第{NN+1}周/
        └── ...
`

示例：
`
news-cards/
├── 2026年第32周/
│   ├── card_01.jpg
│   └── ...
└── 2026年第33周/
    └── ...
`

## 工作流程

### 第 1 步：准备 HTML

1. 从输入目录读取 JSON/Markdown
2. 解析输入数据
3. 组装 HTML 卡片页面：
   - 封面（.card.cover）
   - N 条新闻（.card.news）
   - 末页（.card.thanks）
4. 应用可变参数（颜色、标题、导读等）

### 第 2 步：图片本地化

1. 检查 imgs/ 目录是否已有引用图片
2. 缺失则用 Tavily 搜索文章标题补位
3. 全部转为 JPG 格式
4. 验证 FF D8 合法头

### 第 3 步：高清截图

输出到 {标题}/{YYYY}年第{NN}周/ 目录：

`python
from playwright.sync_api import sync_playwright
import os

out_dir = f"{title}/{year}年第{week}周"
os.makedirs(out_dir, exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(
        executable_path=r"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe",
        args=['--no-sandbox', '--disable-setuid-sandbox']
    )
    page = browser.new_page(
        viewport={"width": 420, "height": 800},
        device_scale_factor=3  # 3x Retina
    )
    page.goto("file:///path/to/cards.html")
    page.wait_for_timeout(2000)
    
    cards = page.query_selector_all(".card")
    for i, card in enumerate(cards, 1):
        card.screenshot(
            path=f"{out_dir}/card_{i:02d}.jpg",
            type="jpeg",
            quality=95
        )
    browser.close()
`

### 第 4 步：验证

- [ ] 12 张 JPG 全部生成
- [ ] 全部 FF D8 合法头
- [ ] 文件大小 > 10KB
- [ ] 输出目录：{标题}/{YYYY}年第{NN}周/

## 常见 Pitfalls

1. **图片链接失效**：来源页 403/404 时，Tavily 搜索补位
2. **截图模糊**：确保 device_scale_factor=3，不能用 1x
3. **圆角残留**：检查 CSS 中 --r:0px 不是 18px
4. **HTML 图片引用错误**：使用相对路径 imgs/xx.jpg，不用绝对路径

## 验证 Checklist

- [ ] 输入解析正确（JSON/Markdown）
- [ ] 可变参数已应用
- [ ] 所有图片本地化且 JPG 格式
- [ ] HTML 渲染无报错
- [ ] 12 张 JPG 导出成功
- [ ] 全部 FF D8 合法
- [ ] 输出目录：{标题}/{YYYY}年第{NN}周/
- [ ] 文件大小合理（90KB-430KB）
