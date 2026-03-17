---
name: frontmatter-shared
description: >
  提供本项目统一的文章 frontmatter 生成与校验规范，供其他写作类 skill 复用。
  关键词：frontmatter、文章头部、统一元数据、博客模板、字段规范。
---

# Frontmatter Shared Skill

## 适用场景

当其他 Skill 需要创建或更新文章 Markdown 头部（frontmatter）时，统一调用本规范。

---

## 标准模板

```yaml
---
title: <文章标题>
description: <1-2 句摘要，建议 50-100 字>
published: <YYYY-MM-DD>
tags: [<标签1>, <标签2>]
category: 原创
draft: false
image: ""
---
```

---

## 字段规则

1. `title`
   - 必填。
   - 优先使用正文一级标题 `# ...` 或来源平台原始标题。

2. `description`
   - 必填。
   - 若源内容未提供，基于正文自动总结 1-2 句。

3. `published`
   - 必填，格式固定为 `YYYY-MM-DD`。
   - 来源优先级：
     1) 来源平台发布日期（如 CSDN 页面发布日期）
     2) 当前日期（如 draft 发布场景）

4. `tags`
   - 必填，建议 2-5 个。
   - 优先来源已有标签，其次正文高频技术关键词。

5. `category`
   - 选填，默认 `原创`。
   - 若来源明确给出分类，可沿用。

6. `draft`
   - 必填，发布到 `src/content/posts` 时固定 `false`。

7. `image`
   - 仅在存在封面图时填写相对路径（如 `./image.png`）。
   - 没有封面图可省略该字段，或保留空字符串。

---

## 输出检查清单

1. frontmatter 使用 YAML 三横线完整包裹。
2. `published` 日期合法且格式正确。
3. `tags` 为数组格式，不是纯字符串。
4. 未出现与正文冲突的重复标题描述。
5. 发布型文章必须 `draft: false`。

---

## 给其他 Skill 的调用约定

- 其他 Skill 在“生成 Frontmatter”步骤中，不再重复写完整模板和规则。
- 仅声明：
  - “frontmatter 生成请遵循 `frontmatter-shared` Skill”
  - 并补充本场景特有的 `published` 来源（例如“取 CSDN 原发布日期”或“取当前日期”）。
