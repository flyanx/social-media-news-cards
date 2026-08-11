# 新闻卡片生成器 · 参考资料

## 常见问题与解决

### 1. 图片防盗链
- Nature、Science、FDA 等机构的图片有 403 防盗链
- 解决方案：Tavily 搜索文章标题 → 获取新闻媒体报道的配图链接
- 检查 magic bytes 排除 Google 验证码页面（HTML 而非图片）

### 2. Tavily 图片过滤
- 过滤 `lookaside`、`fbcdn`、`utm_source` 等社交爬虫链接
- 优先用文章标题搜索而非泛泛的关键词
- `include_images: true` + `max_results: 8`

### 3. PowerShell 语法限制
- PowerShell 不支持单引号字符串内的双引号
- JSON 转义很痛苦，推荐用 Python 脚本调用 curl/requests
- 复杂操作一律用 Python 脚本文件而非 inline -c

### 4. Playwright 浏览器安装
- `playwright install chromium` 可能超时（网络问题）
- 改用 Edge 浏览器：`executable_path=r"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe"`
- `device_scale_factor=3` 实现 3x Retina 输出

### 5. 图片格式统一
- 来源图格式不一（PNG/JPG/WebP）
- 全部用 Pillow 转为 JPG：`Image.save('JPEG', quality=90)`
- 检查 `data[:2] == b'\xff\xd8'` 验证合法 JPEG

## 设计决策记录

| 决策 | 原因 |
|---|---|
| 直角（0px） | 用户偏好，小红书风格 |
| 底部来源 12px | 手机可读性 |
| 封面橙色渐变 | 用户认为黑色不如橙色好看 |
| 朴素标题 | "本周最重磅"等话术太土 |
| 无 emoji | 不够高级 |
| 无下期预告 | 不知道更新什么 |
| 3x DPI 截图 | 手机上看清楚 |

## 文件结构

```
分子生物学每周快报/
├── 2026年第32周/
│   ├── input.json              — 输入数据（JSON/Markdown）
│   ├── xiaohongshu_cards.html  — 生成的卡片 HTML
│   ├── imgs/                   — 本地图片（01.jpg ... 10.jpg）
│   ├── card_01.jpg             — 导出的 JPG
│   ├── card_02.jpg
│   └── ...
├── 2026年第33周/
│   └── ...
```

输入输出在同一目录下：`分子生物学每周快报/{YYYY}年第{NN}周/`

## Tavily 搜索脚本

```python
import urllib.request, json
API_KEY = "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"
payload = json.dumps({
    "api_key": API_KEY,
    "query": "文章标题关键词",
    "max_results": 8,
    "include_images": True,
}).encode("utf-8")
req = urllib.request.Request("https://api.tavily.com/search", data=payload,
                            headers={"Content-Type": "application/json"})
with urllib.request.urlopen(req, timeout=30) as r:
    data = json.loads(r.read().decode("utf-8"))
imgs = [img for img in data.get("images", []) if 'lookaside' not in img]
```

## 截图脚本

```python
from playwright.sync_api import sync_playwright
import os

out_dir = r"F:\IGEBio-Synapse\分子生物学每周快报\2026年第32周"
os.makedirs(out_dir, exist_ok=True)

with sync_playwright() as p:
    browser = p.chromium.launch(
        executable_path=r"C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe",
        args=['--no-sandbox', '--disable-setuid-sandbox']
    )
    page = browser.new_page(viewport={"width": 420, "height": 800}, device_scale_factor=3)
    page.goto("file:///F:/IGEBio-Synapse/xiaohongshu_cards.html")
    page.wait_for_timeout(2000)
    
    for i, card in enumerate(page.query_selector_all(".card"), 1):
        card.screenshot(path=f"{out_dir}/card_{i:02d}.jpg", type="jpeg", quality=95)
    
    browser.close()
```
