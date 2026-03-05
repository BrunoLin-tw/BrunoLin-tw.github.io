# 如何使用文章模板（POST_TEMPLATE.md）

## 1) 複製模板到 `_posts/`

請把 `POST_TEMPLATE.md` 複製成新的文章檔案，並放到 `_posts/` 目錄。

檔名格式必須是：

`YYYY-MM-DD-your-title.md`

範例：

`_posts/2026-03-05-my-first-post.md`

## 2) 修改 Front Matter

打開新文章後，先修改最上方這段：

```md
---
title: "請改成你的文章標題"
date: 2026-03-05 21:30:00 +0800
categories:
  - tech
tags:
  - jekyll
  - notes
---
```

請至少更新：

- `title`
- `date`
- `categories`
- `tags`

## 3) 撰寫文章內容

在 Front Matter 下方用 Markdown 撰寫內容即可。

可使用：

- `##` 小標題
- 清單（`-` 或 `1. 2. 3.`）
- 程式碼區塊（```）
- 圖片（`![alt](路徑)`）

## 4) 提交到 GitHub

```bash
git add _posts/你的文章檔名.md
git commit -m "Add post: 文章標題"
git push
```

推上 GitHub 後，網站部署完成就會自動顯示。

## 5) 快速檢查顯示位置

文章通常會出現在：

- 首頁
- `/posts/`
- 對應的分類頁 `/categories/`
- 對應的標籤頁 `/tags/`
