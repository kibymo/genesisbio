# Genesis Biotech — 維護手冊 (Hugo 版)

> 這個目錄是 **Hugo 靜態站**（取代舊 PHP 版本）。
> 老 PHP 站備份在 `/home/dink/projects/genesisbio/old/_backup_pre-refactor/`，已不再維護。

---

## 1. 5 分鐘上手

### 1.1 啟動 dev server
```bash
cd /home/dink/projects/genesisbio-hugo
hugo server --port 8765 --bind 0.0.0.0
```
開瀏覽器 → `http://localhost:8765/`

### 1.2 正式建置（給主機 deploy）
```bash
hugo --minify
```
產出在 `public/`（96MB，53 頁 HTML + 404 PDFs + 88 圖 + 1 CSS）。

### 1.3 改東西的 3 條鐵則
| 想改的東西 | 改這裡 |
|-----------|--------|
| 某頁文字 | `content/<slug>.md` |
| 選單項目 | `data/navigation.yaml` |
| PDF 位置/檔名 | `data/coa.yaml` |
| 樣式/顏色 | `static/css/style.css` |

---

## 2. 目錄結構

```
genesisbio-hugo/
├── hugo.toml                 # 站設定（title, baseURL, theme）
├── content/                  # 內容頁（48 個 .md，扁平）
│   ├── about.md              # slug = 檔名
│   ├── animal.md
│   ├── antibodies.md
│   └── ... 48 個
├── data/
│   ├── navigation.yaml       # 導航的「單一事實來源」
│   └── coa.yaml              # 294 個 PDF 索引（key → 路徑）
├── themes/genesis/           # 自訂 theme
│   ├── theme.toml
│   ├── layouts/
│   │   ├── _default/
│   │   │   ├── baseof.html   # 全站 wrapper（head, body, footer）
│   │   │   └── single.html   # 內容頁模板
│   │   ├── index.html        # 首頁
│   │   ├── partials/
│   │   │   ├── nav.html      # 12 主選單 + 20 子選單
│   │   │   └── footer.html
│   │   └── shortcodes/
│   │       ├── manifest.html # {{< manifest "key" >}} → 查 coa.yaml
│   │       └── raw.html      # {{< raw >}}...{{< /raw >}} → 跳過 Markdown
│   └── assets/, static/      # theme 自己的資源
├── static/                   # 直接複製到 public/ 的靜態檔
│   ├── COA/                  # 404 個 PDF
│   ├── css/style.css         # 8.3KB 響應式 CSS
│   ├── images/               # 88 個 jpg/gif + 1 spacer.gif
│   └── *.jpg                 # 88 個國家旗/產品圖
└── public/                   # hugo --minify 產出（deploy 這個目錄）
```

---

## 3. 常見維護情境

### 3.1 加新頁面

```bash
# 1. 創 .md
echo -e '---\ntitle: "My New Page"\n---\n\n{{< raw >}}\n<p>內容放這（包 raw 因為有 legacy HTML）</p>\n{{< /raw >}}' > content/my-page.md

# 2. 加進選單（data/navigation.yaml）
# 在想要的位置插入：
#   - { href: my-page.php, label: "My Page", id: my_page }

# 3. 測
hugo server
# http://localhost:8765/my-page/
```

> **為什麼要包 `{{< raw >}}`？** 因為大部分內容頁用 legacy `<TABLE>` HTML，
> 而 Markdown 會把縮排的 `<TABLE>` 當 code block。
> `raw` shortcode 讓 HTML 原文輸出。

### 3.2 改/加/刪 PDF

```bash
# 1. 把新 PDF 丟進
cp /path/to/new.pdf static/COA/

# 2. 編輯 data/coa.yaml
echo "new_key: COA/new.pdf" >> data/coa.yaml

# 3. 頁面裡引用
# 在對應的 .md 裡：
[Download COA]({{< manifest "new_key" >}})
```

**規則**：
- `key` 唯一、英文小寫、底線分隔
- 不需要在頁面直接寫 PDF 路徑，永遠走 `{{< manifest "..." >}}`
- 刪除：從 `coa.yaml` 移除 + 從 `static/COA/` 刪檔

### 3.3 改選單

只動 `data/navigation.yaml`。格式：

```yaml
- href: about.php           # 連結 .php 是歷史相容，模板會自動轉 /about/
  label: About              # 顯示文字
  id: about                 # 內部 id（給 highlight 用）
  sub:                      # 選填：子選單
    - { href: animal.php, label: Animal }
```

加新子選單：在 `sub:` 下面加一行。

### 3.4 改樣式/顏色

`static/css/style.css` 8.3KB，所有 class 看檔頭註解。改完直接 reload。

### 3.5 改站名/版權/描述

`hugo.toml` 的 `[params]` 區段：

```toml
[params]
  description = "..."
  copyright = "Genesis Biotech Inc."
```

---

## 4. URL 與連結慣例

- 站內連結寫 `.php`（`href: about.php`），模板自動轉 `/about/`
- 圖片寫絕對路徑 `/xxx.jpg`（對應 `static/xxx.jpg`）
- PDF 走 shortcode：`{{< manifest "key" >}}`（對應 `data/coa.yaml`）
- 跨語言：footer 的「中文版」連結 `/chn-1/home.htm`（舊版 Big5，暫不動）

> ⚠️ **不要**在 .md 內容裡寫死 `.htm` 或 `/COA/xxx.pdf` 路徑。
> 路徑變了 = 全站壞掉。所有 PDF 一律走 `manifest`。

---

## 5. 部署

`hugo --minify` 產出在 `public/`，整個目錄丟到任何 static host 即可：
- Netlify / Vercel / Cloudflare Pages：把 `genesisbio-hugo/public/` 設為 publish dir
- Apache/Nginx：把 `public/` 整個 rsync 過去
- 本機檔案：直接瀏覽 `public/index.html`

baseURL 在 `hugo.toml` 第一行，部署前要改成實際網址。

---

## 6. 緊急救援

### Q1: 我改了 .md 頁面掛了
```bash
# Hugo server 仍跑著？按 Ctrl+C 停止
# 備份並還原
cp content/broken.md content/broken.md.bak
git checkout content/broken.md  # 若有 git
# 或從 old/_backup_pre-refactor/ 抓原始 .htm 重轉
```

### Q2: hugo build 失敗
看錯誤訊息最末段，常見原因：
- 缺 `---` frontmatter 結尾
- `{{< shortcode >}}` 沒閉合
- `data/*.yaml` 縮排錯誤（YAML 不准用 tab）

### Q3: PDF 404
```bash
# 1. 確認檔案在
ls static/COA/GB-10001.pdf
# 2. 確認 coa.yaml 有對應 key
grep "gb10001" data/coa.yaml
# 3. 確認頁面 .md 引用正確
grep "gb10001" content/animal.md
```

### Q4: 選單連結 404
確認 `data/navigation.yaml` 裡的 `href` 對應到 `content/<slug>.md` 存在。

---

## 7. 與舊 PHP 站對照

| 舊 (PHP) | 新 (Hugo) |
|----------|-----------|
| `eng/includes/header.php` | `themes/genesis/layouts/partials/ + baseof.html` |
| `eng/includes/nav.php` (`$NAV` 陣列) | `data/navigation.yaml` |
| `eng/includes/manifest.php` (`$PDF` 陣列) | `data/coa.yaml` |
| `eng/css/style.css` | `static/css/style.css` |
| `eng/COA/*.pdf` | `static/COA/*.pdf` |
| `eng/*.php`（內容頁） | `content/*.md`（frontmatter + raw HTML） |
| `eng/*.jpg` | `static/*.jpg` |
| `eng/includes/header.php` 的 `<?= manifest('x') ?>` | `{{< manifest "x" >}}` |
| `eng/about.php` 的 `<?php require 'header.php' ?>` | `themes/genesis/layouts/_default/single.html` 自動套 |

---

## 8. 版控建議

`hugo new site` 沒建 git。建議現在 `git init`：

```bash
cd /home/dink/projects/genesisbio-hugo
git init
echo -e 'public/\nresources/\n.hugo_build.lock' > .gitignore
git add . && git commit -m "Initial Hugo migration"
```

之後改東西一律 commit，舊 PHP 站備份在 `old/_backup_pre-refactor/`。

---

## 9. 已知限制

- **chn-1/ 簡中版**：未轉 Hugo，footer 仍連到 `/chn-1/home.htm`（Big5 舊版）
- **search.php**：未處理
- **legacy `<TABLE>` 內容**：HTML 語意差（響應式靠 CSS 勉強撐住），但能用
- **無 sitemap.xml**：Hugo 預設有，可在 `hugo.toml` 加 `[sitemap]`
- **無 favicon**：`themes/genesis/static/favicon.ico` 為空，瀏覽器會 404
