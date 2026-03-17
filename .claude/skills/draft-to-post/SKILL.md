---
name: draft-to-post
description: >
   从 src/content/draft 读取草稿，按项目文章规范快速生成到 src/content/posts。
   自动使用当前日期作为文件名与 published，并复用统一 frontmatter skill。
  关键词：草稿转文章、draft生成posts、快速发文、生成博客、从草稿发布。
---

# Draft To Post Skill

## 触发场景

当用户：
- 说“根据 draft 生成文章 / 发布草稿 / 从草稿创建文章 / 快速创建文章”
- 提到草稿已放在 `src/content/draft/`，希望产出到 `src/content/posts/`

---

## 输入约定

- 草稿目录：`src/content/draft/`
- 默认草稿文件：`src/content/draft/draft.md`
- 若目录下存在多个 `.md`：
  1. 优先使用用户明确指定的文件
  2. 若未指定，使用最近修改的 `.md`

---

## 执行步骤

### Step 1 - 读取草稿并提取结构

1. 读取草稿 Markdown。
2. 识别标题与正文：
   - 若首个一级标题 `# ...` 存在，作为文章标题。
   - 若不存在一级标题，从首段提炼一句短标题。
3. 识别现有 frontmatter（若有）：
   - 可复用字段：`tags`、`category`、`description`、`image`
   - 强制覆盖字段：`published`（使用当前日期）、`draft`（设为 `false`）

---

### Step 2 - 生成发布日期与目标路径

1. 使用当前日期生成：
   - `published`: `YYYY-MM-DD`
   - 文件日期前缀：`YYYYMMDD`
2. 目标文件规则（与现有文章规范一致）：
   - 无图片资源：`src/content/posts/{YYYYMMDD}.md`
   - 同日期冲突：`src/content/posts/{YYYYMMDD}-{keyword}.md`

> `keyword` 使用标题核心英文词或拼音短词，避免中文空格与特殊字符。

---

### Step 3 - 生成 Frontmatter（复用共享 Skill）

- frontmatter 生成必须遵循 `frontmatter-shared` Skill。
- 本场景特例：
   - `published` 强制使用当天日期。
   - `category` 无值时默认 `原创`。
   - 发布到 `posts` 时 `draft: false`。

---

### Step 4 - 正文清洗与规范化

1. 若正文已包含 H1，frontmatter 后保留原 H1，不重复制造第二个等价标题。
2. 补全代码块语言标注（如 `java`、`bash`、`ts`、`json`）。
3. 清理明显噪音内容（无意义分隔线、重复空行、孤立广告文案）。
4. 数学表达使用 KaTeX：
   - 行内：`$...$`
   - 块级：`$$...$$`
5. 外链优先转换为带语义的 Markdown 链接文案，不保留裸 URL。
6. 草稿里面都是作者想保留的内容，不要随意去除，可以换种表现方式或者进行整合。
---

### Step 5 - 写入 posts

1. 创建目标 Markdown 文件。
2. 不删除原草稿文件，默认保留在 `src/content/draft/`。
3. 完成后向用户反馈：
   - 生成的文件路径
   - 使用的发布日期
   - 是否发生重名并使用后缀

---

## 输出示例

输入草稿：`src/content/draft/draft.md`

当前日期：`2026-03-17`

输出文件：`src/content/posts/20260317.md`

头部示例：

```markdown
---
title: BIO 与 NIO 的核心差异
description: 本文从线程阻塞、上下文切换和资源占用三个维度解释 BIO 与 NIO 的本质差异，并结合 Java 示例说明各自适用场景。
published: 2026-03-17
tags: [java, nio, bio, 并发]
category: 原创
draft: false
---
```

---

## 注意事项

- 仅在用户明确要求时才批量处理多个草稿文件。
- 同日期多篇文章必须处理命名冲突，不能覆盖已有文章。
- 若草稿内容过短（如仅有标题），先提示用户补充正文再生成。
- 默认遵循本仓库写作规范，不插入“来源声明”尾注。
- frontmatter 规则统一由 `frontmatter-shared` 维护，避免在本 Skill 重复定义。
