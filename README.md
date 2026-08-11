# social-media-news-cards

> Hermes skill: export HTML cards to HD JPG images for social media. 3x Retina, Playwright screenshot, Xiaohongshu/WeChat format.

将已组装好的 HTML 卡片页面导出为独立 JPG 图片文件，用于小红书/微信公众号等自媒体发布。

## ✨ 特性

- 📱 **手机优先**：400×600px 竖版设计，适配小红书/公众号
- 🔍 **高清输出**：3x Retina 像素密度，1200×1800px 实际分辨率
- 🎨 **灵活配色**：封面颜色、标题、导读均可配置
- 🖼️ **真实配图**：优先本地文件，Tavily 搜索补位，禁用无关图库
- ⚡ **一键导出**：Playwright 自动截图，12 张卡片批量生成
- 🔒 **本地处理**：图片全部本地化，无防盗链问题

## 🚀 快速开始

### 1. 安装

`ash
# 克隆仓库
git clone https://github.com/flyanx/social-media-news-cards.git

# 复制到 Hermes skills 目录
# Windows
xcopy /E /I social-media-news-cards %USERPROFILE%\.hermes\skills\creative\social-media-news-cards

# macOS/Linux
cp -r social-media-news-cards ~/.hermes/skills/creative/social-media-news-cards
`

### 2. 环境依赖

`ash
pip install playwright Pillow pyyaml
playwright install chromium
`

> 如果 Playwright Chromium 下载超时，可改用系统自带的 Edge 浏览器（SKILL 已内置路径）

### 3. 准备输入 JSON

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

### 4. 触发 SKILL

> "导出第32周的卡片"

或：

> "生成周报图片，第32周"

### 5. 获取输出

在 {标题}/{YYYY}年第{NN}周/ 目录下获取 card_01.jpg ~ card_12.jpg

## 📐 输出规格

| 参数 | 值 |
|---|---|
| 尺寸 | 400×600px（竖版） |
| 分辨率 | 3x DPI（Retina，实际 1200×1800px） |
| 格式 | JPEG |
| 质量 | 95% |
| 文件大小 | 90KB ~ 430KB |

## 🎨 设计规范

### 卡片结构

- **封面**：标题 + 周期 + 导读 + 3 条高亮
- **新闻卡片**：编号/图片/标题/摘要/标签/深度解读/底部来源
- **末页**：完

### 字号层级

| 元素 | 字号 |
|---|---|
| 封面大标题 | 34-36px |
| 新闻标题 | 22px |
| 新闻摘要 | 14.5-15px |
| 深度解读 | 13.5px |
| 底部来源 | 12px |
| 标签 | 11px |

### 禁忌

- ❌ emoji
- ❌ SVG 占位图
- ❌ Unsplash 等通用图库
- ❌ "下期预告"
- ❌ 圆角（要直角）
- ❌ 深色模式（默认浅色）

## 🔧 可变参数

| 参数 | 说明 | 示例 |
|---|---|---|
| 封面配色 | 渐变起始/结束色 | #D4A574 → #A16207 |
| 标题文字 | 每期可能不同 | "每周新闻快报" |
| 导读文案 | 每周不同 | "本期关键发现..." |
| 高亮条目 | 3 条，每周不同 | ["新闻1", "新闻2", "新闻3"] |

## 🔗 与其他 SKILL 配合

### 上游：新闻数据搜集

- hermes-intel：全类型通用搜集（含行业/法规/政策/客户/对比）
- synbio-intel：合成生物学技术情报专用

### 完整工作流

`
1. hermes-intel / synbio-intel
   → 搜集新闻，入库到知识库
   
2. kb_manager.py export
   → 导出 Markdown 学习语料
   
3. social-media-news-cards（本 SKILL）
   → 导出 JPG 卡片图片
   
4. 发布到小红书/公众号
`

## ❓ 常见问题

### Q1：截图模糊？

**原因**：device_scale_factor 不是 3

**解决**：确保 Playwright 配置中 device_scale_factor=3

### Q2：图片加载失败？

**原因**：图片路径错误或防盗链

**解决**：
1. 检查 imgs/ 目录是否存在对应图片
2. 检查 JSON 中 image 路径是否正确
3. 确保图片是合法 JPEG（FF D8 头）

### Q3：Playwright Chromium 下载超时？

**解决**：使用系统 Edge 浏览器，SKILL 已内置路径

### Q4：Tavily 搜索也找不到配图？

**解决**：手动下载配图到 imgs/ 目录，SKILL 会优先使用本地文件

### Q5：如何修改封面颜色？

**解决**：在触发 SKILL 时指定，如：

> "导出第32周卡片，封面配色 #D4A574 → #A16207"

## 📁 项目结构

`
social-media-news-cards/
├── SKILL.md              # SKILL 定义（必须）
├── README.md             # 项目说明
├── LICENSE               # MIT 许可证
├── tutorial.md           # 使用教程
└── references/
    └── workflow.md       # 工作流参考
`

## 📄 许可证

MIT License · IGEBio-Synapse
