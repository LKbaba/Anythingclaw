# 结构化提取指南

当 Agent 用 `WebFetch` 抓取页面后，按照以下优先级提取字段。

## 标题（title）

按优先级取第一个存在的值：

1. `<meta property="og:title" content="...">`
2. `<meta name="twitter:title" content="...">`
3. JSON-LD 中的 `"headline"` 或 `"name"`
4. `<title>` 标签
5. 第一个 `<h1>`

## 描述（description）

1. `<meta property="og:description" content="...">`
2. `<meta name="twitter:description" content="...">`
3. `<meta name="description" content="...">`
4. JSON-LD 中的 `"description"`
5. 正文第一段 `<p>`，截断到 200 字以内

## 主图（hero_image）

1. `<meta property="og:image" content="...">`
2. `<meta name="twitter:image" content="...">`
3. JSON-LD 中的 `"image"`（如果是数组取第一个）
4. 正文第一张 `<img>` 且尺寸 > 300x200

**重要**：必须是**绝对 URL**。如果原值是 `/path/to.jpg` 或 `//cdn.example.com/x.jpg`，需要补全：
- `/path/to.jpg` → `https://<origin>/path/to.jpg`
- `//cdn.example.com/x.jpg` → `https://cdn.example.com/x.jpg`

## 其他图片（images）

最多 6 张。从正文 `<img>` 中挑选，跳过：
- 图标（width/height < 100）
- 头像（src 包含 `avatar`、`gravatar`）
- 广告（src 包含 `ads`、`banner`、`tracking`）
- SVG 装饰元素

## 作者（author）

1. `<meta name="author" content="...">`
2. `<meta property="article:author" content="...">`
3. JSON-LD 中的 `"author"`（可能是字符串或对象的 `name` 字段）
4. 带 `rel="author"` 的链接文字
5. 页面中 `.author` / `.byline` 类名的元素

## 标签（tags）

最多 5 个。按优先级：

1. `<meta property="article:tag" content="...">`（可能多个）
2. `<meta name="keywords" content="...">`（逗号分隔）
3. JSON-LD 中的 `"keywords"`
4. 页面中 `.tag` / `.category` / `rel="tag"` 的元素文字

**清洗规则**：去空格、去重、短语限制在 20 字以内。

## 发布日期（published_date）

1. `<meta property="article:published_time" content="...">`
2. JSON-LD 中的 `"datePublished"`
3. `<time datetime="..."> ` 标签
4. 正文中明显的日期文本（如"2026-03-15"）

格式统一成 `YYYY-MM-DD`。

## Favicon

1. `<link rel="icon" href="...">` 或 `<link rel="shortcut icon" href="...">`
2. `<link rel="apple-touch-icon" href="...">`
3. 默认回退：`https://<domain>/favicon.ico`

## 域名（domain）

从 URL 中解析出主机名部分，去掉 `www.` 前缀。

例：`https://www.behance.net/gallery/xxx` → `behance.net`

## JSON-LD 解析

JSON-LD 通常在 `<script type="application/ld+json">` 里，内容是一个或多个 JSON 对象。

常见 `@type` 值：

- `Article` / `NewsArticle` / `BlogPosting`：提取 headline、description、image、author、datePublished
- `Product`：提取 name、description、image、brand
- `CreativeWork`：提取 name、description、image、creator
- `WebPage`：通常是兜底，信息较少

## 特殊站点经验

### Behance / Dribbble

这类 SPA 站点 WebFetch 大概率拿不到动态内容。如果发现返回的 HTML 几乎是空的（只有 `<div id="root"></div>`），直接标记为失败，原因写"SPA 站点，需要浏览器渲染"。

### Medium / Substack / Ghost 博客

OG 标签非常完整，按标准流程走即可。

### Notion 公开页

`og:image` 经常是 Notion 的通用封面，不是文章实际主图。可以考虑从正文找第一张真实图片。

### 国内站点（知乎、小红书、微信公众号）

反爬较强，WebFetch 可能返回登录墙或验证页。如果响应体中出现"请登录"、"访问太频繁"、"安全验证"等字样，标记失败。

## 建筑设计扩展字段

以下字段是可选的，仅当页面内容明确包含相关信息时提取。**不要编造**。

### 项目类型（project_type）

从页面标题、标签、正文中识别：
- 关键词映射：`住宅`/`residence`/`house` → "住宅"
- `办公`/`office`/`commercial` → "商业"
- `学校`/`school`/`museum`/`library`/`hospital` → "公共"
- `公园`/`park`/`landscape`/`garden` → "景观"
- `室内`/`interior` → "室内"
- `装置`/`installation`/`pavilion` → "装置"

### 项目地点（location）

1. JSON-LD 中的 `"contentLocation"` 或 `"locationCreated"`
2. 正文中明显的地理位置提及（如 "位于深圳南山区"、"Located in Shenzhen"）
3. ArchDaily/gooood 等站点通常在项目信息栏有标准化的地点字段

### 建筑师/事务所（architect）

1. 页面中 `建筑师`/`Architect`/`设计团队`/`Design Team` 标签后的文本
2. gooood/ArchDaily 等站点的项目信息栏
3. `<meta>` 中的 author 信息（对建筑媒体通常是事务所名）

### 材料标签（materials）

最多 5 个。从以下位置提取：
1. gooood 文末的 "材料" 信息栏
2. ArchDaily 的 "Materials" 标签
3. 正文中明确提到的建筑材料名称

常见材料关键词：清水混凝土、玻璃幕墙、木饰面、钢结构、石材、铝板、砖、竹、膜结构

### 建成年份（year）

1. 项目信息栏中的 "年份"/"Year" 字段
2. 正文中 "建成于 20XX 年" / "Completed in 20XX"
3. 与 `published_date` 不同——published_date 是文章发布时间，year 是建筑建成时间

## agent-browser 模式下的提取

当使用 agent-browser 抓取时，提取方式有所不同：

### 从 snapshot 提取

`agent-browser snapshot` 返回精简的 Ref 列表（如 `@e1: heading "标题"`），非常紧凑。从中提取：
- 标题：通常是最显眼的 heading 元素
- 描述：正文前几段文本
- 作者/建筑师：查找 "by"、"architect"、"设计" 等标签附近的文本
- 标签：查找 tag/category 区域的文本

### 从 eval 提取

通过 `agent-browser eval` 执行 JS 来精确提取：

```bash
# 提取所有图片 URL
agent-browser eval "Array.from(document.querySelectorAll('img')).map(img => img.src).filter(src => src && !src.includes('avatar') && !src.includes('icon')).slice(0, 10)"
```

```bash
# 提取 OG 标签
agent-browser eval "JSON.stringify(Object.fromEntries(Array.from(document.querySelectorAll('meta[property^=\"og:\"]')).map(el => [el.getAttribute('property'), el.getAttribute('content')])))"
```

### 截图作为 hero_image 的替代

当页面图片有防盗链、或无法提取到有效的 hero_image URL 时：
1. 使用 `agent-browser screenshot --full output/screenshots/xxx.png` 截取整页（路径直接跟在后面，不用 `-o`）
2. 截图保存到 `output/screenshots/`
3. 在卡片数据中设置 `"screenshot": "screenshots/xxx.png"` 作为替代展示

## 抓不到时的失败原因分类

在生成卡片的 `error` 字段中用以下标准表述，便于用户理解：

- `"HTTP 403 / 拒绝访问"` — 服务器拒绝
- `"HTTP 404 / 页面不存在"` — URL 失效
- `"HTTP 超时"` — 请求超时
- `"SPA 站点，需要浏览器渲染"` — 只拿到了空壳 HTML（仅 WebFetch 模式）
- `"需要登录"` — 返回了登录墙
- `"触发反爬验证"` — 遇到 Cloudflare / 验证码
- `"页面内容为空"` — 响应 200 但提取不到任何有效字段
- `"浏览器导航失败"` — agent-browser 无法打开页面
- `"未知错误：<简短描述>"` — 其他情况
