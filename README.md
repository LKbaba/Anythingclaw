# Anythingclaw

> 一把"什么都能抓回来"的爪子——把 URL 列表变成可浏览的 HTML 画板。

给 Claude Code 用的 Skill。粘一组网址进去，Agent 会逐个访问、提取标题/图片/摘要，再拼成一个瀑布流画板自动打开给你看。

特别适合：

- **建筑设计师** 收集项目灵感、整理参考图
- **设计师** 汇总 Behance/Dribbble 作品
- **研究员** 批量整理资料
- **产品经理** 调研竞品

---

## 两种抓取模式

| 模式 | 能力 | Token 消耗 | 需要安装 |
|------|------|-----------|---------|
| **agent-browser**（推荐） | 能渲染 SPA、能截图、能执行 JS | 极低 | 需要 |
| **WebFetch**（兜底） | 只能抓静态 HTML / SSR 页面 | 低 | 不需要 |

Agent 会自动检测你的环境：有 agent-browser 就用它，没有就用 WebFetch。

**为什么选 agent-browser 而不是 Playwright MCP？**
- Vercel 出品，Rust 原生，专为 AI Agent 设计
- Token 消耗比 Playwright MCP 少 80-94%（snapshot 返回精简 Ref 列表，而非巨大的无障碍树）
- 直接 Bash 调用，不需要跑 MCP Server、不需要改配置文件
- 装好即用，无需重启 Claude Code

---

## 安装

### 第 1 步：安装 Skill

**方式 A：全局安装（推荐，所有项目都能用）**

```bash
# macOS / Linux
git clone https://github.com/LKbaba/Anythingclaw.git ~/.claude/skills/anythingclaw

# Windows（Git Bash）
git clone https://github.com/LKbaba/Anythingclaw.git "$USERPROFILE/.claude/skills/anythingclaw"
```

**方式 B：项目级安装（只在某个项目生效）**

```bash
cd <你的项目根目录>
mkdir -p .claude/skills
git clone https://github.com/LKbaba/Anythingclaw.git .claude/skills/anythingclaw
```

**方式 C：手动下载**

1. 下载本仓库的 ZIP
2. 解压，重命名文件夹为 `anythingclaw`
3. 放到 `~/.claude/skills/` 下（全局）或项目的 `.claude/skills/` 下（项目级）

### 第 2 步：安装 agent-browser（推荐，非必须）

agent-browser 让 Agent 拥有真正的浏览器，可以渲染 SPA、截图、执行 JS。

```bash
npm install -g agent-browser
agent-browser install
```

> `agent-browser install` 会自动下载 Chrome for Testing。如果你本地已有 Chrome/Brave/Playwright 浏览器，它会自动检测。

安装后**立即可用**，无需重启 Claude Code。

> **不装也能用**：没有 agent-browser 时 Agent 会自动降级到 WebFetch 模式，只是对 SPA 站点（Behance、Dribbble 等）的抓取能力会受限。

#### 验证安装

```bash
agent-browser --version
```

能看到版本号就说明装好了。

---

## 使用

装好 Skill 之后，直接和 Claude Code 说话就行：

**最简单的用法**：

```
帮我抓这几个网址：
https://archdaily.cn/cn/xxx
https://gooood.cn/yyy.htm
https://dezeen.com/2026/01/zzz/
```

**从文件读**：

```
urls.txt 里有 20 个设计网站，帮我跑一遍做成画板
```

**带具体目标**：

```
我想做一个"深圳公共建筑"的灵感库，这些是我挑的网站：...
```

**抓 SPA 站点**（需要 agent-browser）：

```
帮我抓这几个 Behance 作品集
```

Claude 会自动识别到 Anythingclaw Skill，按工作流执行，最后弹出一个浏览器画板。

---

## 输出在哪里

生成的 HTML 画板保存在 Skill 目录的 `output/` 下：

```
<skill 目录>/output/gallery-20260424-1030.html
<skill 目录>/output/screenshots/    # agent-browser 截图（如果有）
```

每次任务生成一个新文件，不会覆盖历史。想清理就直接删掉 `output/` 里的文件。

---

## 能抓什么、抓不到什么

### agent-browser 模式

**能抓**：
- 静态 HTML + SSR 页面（和 WebFetch 一样）
- SPA 站点（Behance、Dribbble、Divisare 等）
- 懒加载的图片
- 可截图保存页面视觉

**抓不到**：
- 需要登录的内容
- 被 Cloudflare 等完全阻断的页面

### WebFetch 模式

**能抓**：
- 静态 HTML 页面
- SSR 站点（大多数博客、新闻、产品官网）
- 带 Open Graph 标签的页面

**抓不到**（会如实标注失败原因，不会编造）：
- 纯客户端渲染的 SPA
- 需要登录的页面
- 有强反爬的站点
- 国内社交平台（知乎、小红书、微信公众号等）

---

## 建筑设计师专属功能

Anythingclaw 内置了建筑/设计行业的站点策略和提取规则：

- **22 个常用站点的抓取策略**（ArchDaily、gooood、Dezeen、有方……）
- **建筑扩展字段**：项目类型、地点、建筑师/事务所、材料标签、建成年份
- **图片优先的画板布局**：大图瀑布流，适合视觉导向的工作方式

---

## 目录结构

```
anythingclaw/
├── SKILL.md                          # Skill 主指令（Agent 读这个）
├── README.md                         # 你正在看的这个文件
├── CLAUDE.md                         # Claude Code 项目配置
├── assets/
│   └── gallery_template.html         # 画板 HTML 模板
├── references/
│   ├── extraction_guide.md           # 字段提取规则
│   └── architecture_sites.md         # 建筑设计师常用站点及抓取策略
└── output/                           # 生成的画板和截图（.gitignore 已排除）
    └── screenshots/                  # agent-browser 截图
```

---

## 常见问题

**Q：需要 API Key 吗？**
不需要。Anythingclaw 不调用任何外部 API。

**Q：需要装 Python 吗？**
不需要。但如果你要用 agent-browser（推荐），需要有 Node.js 18+ 环境。

**Q：画板里的图显示破损怎么办？**
大概率是目标网站的防盗链。如果装了 agent-browser，Agent 会自动截图作为替代。

**Q：能抓小红书/微信公众号吗？**
不能。这些平台的反爬机制极强，Web 端几乎无法获取有效内容。Agent 会如实报告失败原因。

**Q：Skill 不生效？**
- 确认目录结构正确，`SKILL.md` 文件在 skill 文件夹的根部
- 重启 Claude Code
- 确认 Claude Code 版本支持 Skills 特性

**Q：agent-browser 装了但不工作？**
- 运行 `agent-browser --version` 确认安装成功
- 运行 `agent-browser install` 确认浏览器已下载
- 确认 Node.js >= 18

---

## License

MIT
