---
name: csdn-import
description: >
  将用户在 CSDN 上发布的博客文章，通过链接抓取并转换为本项目规范格式，
  包含正确的 frontmatter、本地化图片资源，以及来源声明。
  关键词：csdn、CSDN博客、导入CSDN、存到本地、CSDN转换、CSDN文章、保存博客。
---

# CSDN Import Skill

## 触发场景

当用户：
- 提供一个 `blog.csdn.net` 开头的 URL
- 说"把我的 CSDN 博客存到本地 / 导入 CSDN 文章 / 同步 CSDN 博客"等类似意图

---

## CSDN 页面特征说明

### 图片 CDN 域名

CSDN 正文图片通常托管在以下域名，抓取时需识别：

- `https://i-blog.csdnimg.cn/direct/...`（新版上传图片）
- `https://img-blog.csdnimg.cn/...`（旧版上传图片）
- `https://img-blog.csdn.net/...`（更早版本）

### 需要清理的噪音内容

`fetch_webpage` 抓取的页面中包含大量非正文内容，整理正文时必须删除：

- 页面导航栏、面包屑
- 广告区块（`万维广告联盟`、`AI写代码` 操作按钮等文字）
- 右侧侧边栏（`TA的精选`、`大家在看`、`TA的历史创作历程`）
- 评论区、点赞/收藏/分享按钮文字（如 `9`、`13`、`0` 等孤立数字）
- 登录提示（`登录后您可以享受以下权益`）
- 代码块旁的 CSDN 特有标注（`AI写代码 java 登录复制运行`、`登录复制` 等）
- 底部版权备案信息

### 发布日期提取

CSDN 文章发布时间格式通常为：
```
原创于 YYYY-MM-DD HH:mm:ss 发布
```
提取日期部分 `YYYY-MM-DD` 作为 `published` 字段，并以 `YYYYMMDD` 作为文件夹名。

---

## 执行步骤

### Step 1 — 抓取页面

使用 `fetch_webpage` 工具抓取用户提供的 CSDN URL，提取：

1. **文章标题**：页面 `<h1>` 或 `<title>` 中的标题
2. **发布日期**：页面中 `原创于 YYYY-MM-DD` 格式的时间信息
3. **正文 Markdown**：去除上述噪音后的纯正文内容
4. **所有正文图片 URL**：识别 `i-blog.csdnimg.cn` / `img-blog.csdnimg.cn` 图片链接

---

### Step 2 — 处理图片

1. 统计正文中所有图片（`![...](url)` 或 HTML `<img>`）。
2. 命名规则：
   - 单张图片 → `image.png`
   - 多张图片 → `image-0.png`、`image-1.png`、`image-2.png`…（按正文出现顺序）
3. 使用 `run_in_terminal` 执行 `Invoke-WebRequest` 下载（Windows 环境）：

```powershell
Invoke-WebRequest -Uri "<图片原始URL>" -OutFile "c:\...\src\content\posts\<YYYYMMDD>\image-0.png"
```

> 若图片下载失败（403/404），在正文中保留原始 URL 并在末尾告知用户手动替换。

4. 将正文中所有图片引用替换为本地相对路径：
   - `![描述](./image.png)`
   - `![描述](./image-0.png)`

---

### Step 3 — 生成 Frontmatter

```yaml
---
title: <文章标题，保留原中文标题，不修改>
description: <1-2 句话概括核心内容，约 50-100 字，中文>
published: <YYYY-MM-DD，从页面提取的发布日期>
tags: [<标签1>, <标签2>]   # 从文章主题提取 2-5 个精准标签
category: 原创              # CSDN 发布的自己的文章默认为"原创"
draft: false
image: ""                   # 若有封面图填写 ./image.png，否则省略
---
```

**标签提取规则**：
- 优先使用 CSDN 页面中已标注的文章标签（`#哈希算法 #java #HashMap` 等）
- 补充文章中高频出现的核心技术关键词
- 数量控制在 2-5 个

### Step 4 — 整理正文格式

1. **标题层级**：保留原文 H1 标题在正文开头；正文各节从 H2 开始。
2. **代码块语言标注**：CSDN 代码块常缺失语言标注，根据内容补全，如 ` ```java `、` ```bash `、` ```sql ` 等。
3. **清理 CSDN 特有文字垃圾**：删除代码块附近的 `AI写代码 java 登录复制运行`、`登录复制` 等字样。
4. **数学公式**：若正文含数学表达式，转换为 KaTeX 格式（行内用 `$...$`，块级用 `$$...$$`）。
5. **表格**：检查并补全 Markdown 表格对齐线。
6. **去除多余空行**：连续超过 2 个空行的地方压缩为 1 个空行。
7. **不添加分隔线**：正文各章节之间**不要**插入 `---` 水平分隔线，直接用标题分隔即可。
8. **外链展示优化**：正文中出现的裸 URL 统一替换为描述性 Markdown 链接，根据域名和上下文语义推断合适的描述文字，禁止输出纯 URL 作为链接文本。
   - 示例：`[在知乎阅读 cc-switch 完整介绍](https://zhuanlan.zhihu.com/p/xxx)`
   - 示例：`[前往 GitHub 仓库](https://github.com/xxx)`
9. **Bilibili 视频内嵌**：若正文中包含 Bilibili 视频链接（`bilibili.com/video/BVxxx`），提取 BV 号，替换为 iframe 内嵌播放器：
   ```html
   <iframe src="//player.bilibili.com/player.html?bvid={BVID}&autoplay=0" scrolling="no" frameborder="no" framespacing="0" allowfullscreen="true" width="100%" height="500"></iframe>
   ```

---

### Step 5 — 创建文件结构

> 不需要在正文末尾添加来源声明，直接结束即可。

**有图片时**（必须建子文件夹）：

```
src/content/posts/{YYYYMMDD}/
├── {YYYYMMDD}.md
├── image-0.png
└── image-1.png
```

**无图片时**（可直接放在 posts/ 下）：

```
src/content/posts/{YYYYMMDD}.md
```

**同日期多篇文章时**：若 `{YYYYMMDD}.md` 已存在，使用文章主题关键词作为后缀，例如：

```
src/content/posts/{YYYYMMDD}-{keyword}.md
```

示例：`20260310-claudecode.md`

操作顺序：
1. `run_in_terminal` 创建文件夹（有图片时）：
   ```powershell
   New-Item -ItemType Directory -Force "c:\...\src\content\posts\<YYYYMMDD>"
   ```
2. `run_in_terminal` 下载所有图片（见 Step 2）
3. `create_file` 创建 `{YYYYMMDD}.md`

---

## 完整示例

### 输入


> 用户："新建博客 https://blog.csdn.net/2301_79742544/article/details/158657400"

### 执行过程

1. `fetch_webpage` 抓取页面，提取发布日期 `2026-03-04`，文件夹命名 `20260304`
2. 识别正文中 2 张图片：
   - `https://i-blog.csdnimg.cn/direct/562bb787fb6246d6875b0b6f02a1b88f.png`
   - `https://i-blog.csdnimg.cn/direct/2883ccfbad894883a5ef3edf8a71297c.png`
3. 创建文件夹 `src/content/posts/20260304/`
4. 下载为 `image-0.png`、`image-1.png`
5. 生成 `20260304.md`，图片引用替换为 `./image-0.png`、`./image-1.png`

### 输出结构

```
src/content/posts/20260304/
├── 20260304.md
├── image-0.png
└── image-1.png
```

### 输出文件头部

```markdown
---
title: HashMap 中如何用位运算来提高性能
description: 在 Java 开发中，HashMap 是最常用的数据结构之一。本文深入分析为什么容量必须是 2 的幂，以及如何使用位运算替代取模运算，揭示 HashMap 高性能设计背后的底层逻辑。
published: 2026-03-04
tags: [java, HashMap, 哈希算法, 位运算]
category: 原创
draft: false
---
```

## 注意事项

- CSDN URL 中的 `?spm=...` 参数可忽略，直接用文章 ID 部分即可访问。
- 部分 CSDN 文章需要登录才能查看完整内容，若 `fetch_webpage` 返回内容不完整，告知用户手动粘贴正文。
- `published` 严格使用页面上的**原始发布日期**，不使用抓取当天日期。
- 文件夹和文件名统一使用 `YYYYMMDD` 格式（8 位纯数字），与同日期对应。
- 不修改原文标题，保持与 CSDN 上一致。
