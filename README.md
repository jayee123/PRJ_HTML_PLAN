# Buy託了AI｜日韓藥妝智慧推薦系統

> 整合官方商品 API 與多平台社群評論，透過情緒分析與 RAG 向量搜尋，依照膚質、預算、需求精準推薦日韓藥妝。

**類型**：個人・資料工程 + AI 推薦  
**月費**：NT$0（全免費方案）  
**企劃書線上瀏覽**：[drugstore_plan.html](https://jayee123.github.io/PRJ_HTML_PLAN/drugstore_plan.html)

---

## 目錄

1. [專題動機](#1-專題動機)
2. [目的](#2-目的)
3. [方法](#3-方法)
4. [系統需求](#4-系統需求)
5. [進行方式](#5-進行方式)
6. [預期達成目標](#6-預期達成目標)
7. [補充資料](#7-補充資料)

---

## 1. 專題動機

根據交通部觀光署觀光統計資料庫 2026 年 1 月至 4 月統計，日本位居出境旅遊目的地第一、韓國第三，日韓合計占比近五成。藥妝為旅客購物清單首選品項，但消費者面臨三大痛點：

| 痛點 | 說明 |
|------|------|
| 藥妝是旅遊必買品項 | 因價格優惠、種類豐富，赴日韓旅客購物清單首位 |
| 資訊分散於多平台 | 需耗費大量時間於 PTT、Dcard、部落格、影音平台分別搜尋 |
| 難以個人化篩選 | 面對琳瑯滿目商品，難以同時依功能、評價、價格及適用對象快速決策 |

**核心痛點**：消費者購買日韓藥妝時，必須自行整合散落於多個平台的資訊，再手動篩選符合自身條件的商品，耗時且容易做出不符需求的購買決策。

---

## 2. 目的

開發「**Buy託了AI**」——結合 AI 推薦技術與藥妝資訊整合，協助消費者快速找到符合需求的日韓藥妝，打造一站式智慧購物輔助工具。

1. **整合多平台評論，AI 彙整分析**：整合 PTT、Dcard 等評價與使用心得，利用 AI 進行資料彙整，整理各產品優缺點與用戶回饋。
2. **建立完整商品資料庫（官方 API）**：Rakuten API（日本）+ Olive Young 爬蟲（韓國）；@cosme 補充日本口碑評分、화해 補充韓國口碑評分與成分安全評級。
3. **個人化推薦條件篩選**：依旅遊國家、預算及個人需求，提供個人化推薦結果。
4. **可信評論來源設計**：PTT（推文社群審查）+ Dcard（實名認證，每日一篇），結構性降低業配文影響。
5. **一站式智慧旅遊購物工具**：涵蓋日韓兩國藥妝的統一推薦平台。

---

## 3. 方法

### 3-1 各資料來源爬取方式

| 資料來源 | 爬取工具 | 關鍵技術挑戰 | 已驗證筆數 |
|----------|----------|--------------|------------|
| **Rakuten Ichiba API**（日本商品） | 官方 REST API v2026-04-01 | 新版需同時傳 `applicationId`（UUID）與 `accessKey`（pk_ 開頭）；動態 IP 須設 IP 白名單全開 | ✅ 1,116 筆 |
| **Olive Young 爬蟲**（韓國商品） | requests + BeautifulSoup；JS 頁改 Playwright | 靜態分類頁以 BS4 解析；需設 User-Agent 避免 403；成分表在商品詳細頁需另爬 | ✅ 216 筆 |
| **@cosme 排名爬蟲**（日本口碑） | requests + BeautifulSoup，SSR HTML | 嘗試多個 API endpoint 全數 404；改爬 `dl.clearfix` SSR HTML 成功；deep-translator 日文→繁中 | ✅ 480 筆（24 分類） |
| **화해 排名爬蟲**（韓國口碑） | requests + JSON，Gateway API | 探測多組 endpoint 後找到正確 Gateway API；deep-translator 韓文→繁中 | ✅ 860 筆 |
| **PTT MakeUp 板** | requests + BeautifulSoup | `over18=1` Cookie 繞過成年確認；70+ 日韓品牌關鍵字清單篩選相關文章 | ⚠️ 11 筆（metadata，文章內文待爬） |
| **Dcard 美妝版** | Playwright（headless=False） | Cloudflare Turnstile 需真實瀏覽器；攔截 `service/api/v2/posts` XHR 直接取 JSON | 規劃中（第 3 週） |

### 3-2 資料欄位完整度

| 來源 | 已有欄位 | 缺少 / 待補 |
|------|----------|-------------|
| Rakuten API | 商品名(日)、品牌、JPY 售價、圖片 URL、商品說明、評分、評論數 | 成分表（含在商品說明中，需 regex 解析） |
| Olive Young | 商品名(韓/中)、品牌(韓/中)、KRW 售價、圖片 URL、商品連結 | 成分表、評分（需爬詳細頁） |
| @cosme | 大分類(日/中)、品牌(日/中)、商品名(日/中)、評分、口コミ數、價格、圖片 | — 完整 |
| 화해 | 商品名(韓/中)、品牌(韓/中)、平均評分、評論數、KRW 售價、容量、圖片 URL | — 完整 |
| PTT | 標題、作者、日期、推文數、連結 | 文章內文（情緒分析核心，需補爬） |

**主要缺口**：① 成分表需另爬商品詳細頁　② PTT 文章內文尚未爬取　③ Dcard 尚未實作

### 3-3 跨來源資料整合

- **商品配對**：rapidfuzz 模糊比對（中/日/韓）+ 品牌別名對照表 + Olive Young ↔ 화해 以商品 ID 對齊
- **多語言處理**：@cosme 日文→繁中、화해 韓文→繁中（deep-translator）；PTT/Dcard 繁中直接使用

### 3-4 立場分析

```
模型：uer/roberta-base-finetuned-jd-binary-chinese（約 400MB，CPU 可跑）
輸出：positive / negative
用途：計算每個商品的好評率（sentiment_pos_rate），作為推薦排序核心分數
```

訓練資料為中文電商商品評論（京東），對藥妝評語準確率最高；HuggingFace pipeline，`device=-1` 跑 CPU。

### 3-5 RAG 向量搜尋

**嵌入模型**：`sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`（約 120MB，CPU 可跑，原生支援中/日/韓）

**向量資料庫**：ChromaDB（`PersistentClient`，本機 `./chroma_db`，零設定）

| 步驟 | 說明 |
|------|------|
| ① 建立商品向量索引 | 向量文字 = `商品名 + 品牌 + 分類 + 成分前 5 項`；`model.encode()` 批次向量化 → `col.upsert()` 寫入 ChromaDB |
| ② 使用者查詢 → 語意推薦 | 自然語言輸入 → encode → `col.query()` Top-K → 結合好評率計算最終推薦分數 |
| ③ 評論 ↔ 商品配對 | 評論文字（標題 + 內文前 300 字）向量化 → 搜尋最近似商品；相似度門檻 **0.65** |

### 3-6 資料清洗

| 清洗步驟 | 實作方式 | 狀態 |
|----------|----------|------|
| 商品去重 | `rakuten_id` / 商品 URL 為唯一 key，`seen = set()` 過濾 | ✅ 已實作 |
| 空值 / 空白過濾 | `if not text or not text.strip(): return` | ✅ 已實作 |
| 文字長度截斷 | 立場分析 `text[:512]`；評論配對 `text[:300]`；Rakuten `itemCaption[:200]` | ✅ 已實作 |
| HTML 標籤去除 | BeautifulSoup `.get_text(strip=True)` | ✅ 已實作 |
| CSV 編碼統一 | 所有爬蟲輸出 `utf-8-sig`（Excel 可直開） | ✅ 已實作 |
| 翻譯空字串防呆 | 送翻譯前先判斷 `if not text.strip()` | ✅ 已實作 |
| 成分表結構化解析 | Rakuten 需 regex；Olive Young 需另爬詳細頁 | ⚠️ 初步規劃 |
| 跨平台商品名稱正規化 | 品牌別名對照表 + rapidfuzz | ⚠️ 初步規劃 |

---

## 4. 系統需求

### 4-1 功能需求

| # | 功能 | 技術方案 | 優先度 |
|---|------|----------|--------|
| 1 | 商品資料擷取 | Rakuten API + Olive Young 爬蟲 | P1 |
| 2 | 評論語料爬取 | PTT / Dcard / @cosme / 화해 | P1 ✅ |
| 3 | 評論立場分析 | uer/roberta → 好評率 | P1 |
| 4 | 評論與商品配對 | rapidfuzz + RAG，門檻 0.65 | P1 |
| 5 | RAG 語意推薦 | sentence-transformers + ChromaDB | P1 |
| 6 | 結構化篩選 API | FastAPI + MongoDB，膚質 × 品類 × 預算 | P1 |
| 7 | 前端展示介面 | HTML / CSS / JS，篩選面板 + 商品卡片 | P1 |
| 8 | 商品 LLM 摘要 | GPT-4o / Groq API | P2 |
| 9 | YouTube 語音轉文字 | Faster-Whisper（Colab GPU） | P2 |
| 10 | 趨勢排行榜 | MongoDB 增量查詢 | P3 |

### 4-2 系統架構

```
資料輸入          →   處理層              →   儲存 + RAG           →   展示層
─────────────────────────────────────────────────────────────────────────────
Rakuten API           立場分析                MongoDB Atlas            FastAPI
Olive Young 爬蟲      (uer/roberta)           (商品 + 評論)             /recommend API
PTT (requests/BS4)    商品名稱對齊             ChromaDB                 Render.com
Dcard (Playwright)    (rapidfuzz)             (向量索引)               靜態 HTML/JS
@cosme (BS4+翻譯)     評分計算                sentence-transformers    GitHub Pages
화해 (JSON API+翻譯)  GPT-4o 摘要(P2)         Upstash Redis(快取)
```

### 4-3 費用（全免費方案）

| 項目 | 工具 | 月費 |
|------|------|------|
| 日本商品資料 | Rakuten Ichiba API（3 萬次/日） | NT$0 |
| 韓國商品資料 | Olive Young 爬蟲 | NT$0 |
| 日本口碑評分 | @cosme 爬蟲 + deep-translator | NT$0 |
| 韓國口碑評分 | 화해 爬蟲 + deep-translator | NT$0 |
| 主資料庫 | MongoDB Atlas M0（512MB） | NT$0 |
| 向量資料庫 | ChromaDB 本機 | NT$0 |
| 後端部署 | Render.com（750h/月） | NT$0 |
| 前端部署 | GitHub Pages | NT$0 |
| GPU 轉文字 | Google Colab + Kaggle | NT$0 |
| GPT-4o 摘要（P2，一次性） | OpenAI API | 約 NT$30–60 |
| **合計** | | **NT$0 / 月** |

---

## 5. 進行方式

### 5-1 開發時程（8 週）

| 週次 | 里程碑 | 主要任務 |
|------|--------|----------|
| 第 1 週 | 環境建置 | Rakuten API 申請、Olive Young 爬蟲、MongoDB Atlas 建立、Schema 設計 |
| 第 2 週 | 商品資料批次擷取 | 日系 300 SKU、韓系 200 SKU、品牌別名表、rapidfuzz 去重 |
| 第 3 週 ✅ | 評論語料爬取 | PTT / Dcard / @cosme / 화해 全部完成 |
| 第 4 週 | 立場分析 + 評分計算 | uer/roberta 跑全部評論、好評率計算、人工抽查 |
| 第 5 週 | RAG 向量化 | ChromaDB 建索引、評論 ↔ 商品配對、查詢效果測試 |
| 第 6 週 | FastAPI 推薦系統 | `/recommend` API、Redis 快取、壓力測試 |
| 第 7 週 | 前端整合 + 部署 | 接真實 API、Render.com + GitHub Pages 上線 |
| 第 8 週 | P2 加分 + 簡報 | GPT-4o 摘要、Whisper 影片轉錄、Demo 影片準備 |

### 5-2 評審問題預演

**Q：商品資料完整嗎？你怎麼取得日韓商品資料？**

商品完整資料（名稱、圖片、成分、售價）用官方管道取得：日本用 Rakuten Ichiba API（每日 3 萬次免費）、韓國用 Olive Young 爬蟲（商品頁含名稱、品牌、售價、圖片、成分表）。另搭配 @cosme 爬蟲補充日本口碑評分，화해 爬蟲補充韓國口碑評分 + 成分安全評級，形成「@cosme（日）↔ 화해（韓）」對稱架構。

**Q：你的評論是真實用戶還是業配？怎麼保證？**

選擇「結構性可信」的平台，不試圖辨識業配文：① PTT 社群推文機制自我審查，具公信力；② Dcard 實名認證每日一篇，刷評論成本極高。刻意不用 Instagram（反爬強 + 業配辨識困難）。

**Q：RAG 是必要的嗎？直接用關鍵字搜尋不行嗎？**

有兩個具體原因：① 評論配對——PTT 說「這款凝凍超補水」，關鍵字找不到「肌ラボ極潤」，向量搜尋可跨語言語意配對；② 使用者查詢——「不黏膩的保濕」可匹配到「水感質地」「清爽不油」等語意相關商品。

**Q：一個人做得完嗎？8 週夠嗎？**

已在本機完成 Demo 驗證（20 個商品 + 50 篇評論跑通完整流程）。P1 核心功能第 7 週完成，第 8 週為加分項目。Whisper 轉文字規劃用 Colab 免費 GPU，不依賴本機算力。

---

## 6. 預期達成目標

### 6-1 核心功能目標

| 目標 | 說明 |
|------|------|
| 完整商品資料庫 | 500+ SKU，日韓商品各含圖片、成分表、售價；搭配 @cosme + 화해 口碑評分 |
| 全評論立場分析 | 5,000+ 篇評論完成 positive/negative 標記，輸出好評率並可視化 |
| RAG 語意推薦 | 自然語言描述需求 → 向量搜尋最相關商品，可現場操作展示 |
| 三維度結構化篩選 | 膚質 × 品類 × 預算，篩選結果即時更新 |

### 6-2 期末交付物

| 交付項目 | 說明 | 狀態 |
|----------|------|------|
| 前端展示介面 | 篩選面板 + 商品卡片（含圖片 + 分數）+ 評論來源標示 | P1 確定交付 |
| 商品資料庫 | 500+ SKU，日韓各含圖片與成分表 | P1 確定交付 |
| 評論立場分析 | 5,000+ 篇評論完成，好評率可視化 | P1 確定交付 |
| RAG 語意搜尋 | 使用者描述需求，系統找出最相關商品 | P1 確定交付 |
| AI 圖片幻覺驗證報告 | GPT-4o 直接回傳圖片 URL 幻覺率測試（0/5）說明為何改用官方 API | ✅ 已完成 |
| 商品 LLM 摘要 | 500 個商品各 2–3 句 GPT-4o 推薦摘要 | P2 時間許可 |
| Whisper 影片轉文字 | 300 支無字幕 YouTube 影片，Colab GPU 轉錄 | P2 時間許可 |

### 6-3 質化成果

使用者不再需要整合 5 個平台的資訊、自行判斷評論真假。輸入膚質和預算，系統在 1 秒內回傳有社群好評率支撐的推薦，每個推薦標示「來自幾個平台、幾則評論計算」，透明可驗證。

> **本專題每個技術決策都有數據支撐**——GPT-4o 圖片幻覺率（0/5 驗證）、選擇 PTT 而非 IG 的可信度論據、使用官方 API 而非純爬蟲的完整性論證。這不是純規劃，而是有驗證過的實作。

---

## 7. 補充資料

### 爬蟲技術細節

#### PTT MakeUp 板
- `requests` + `BeautifulSoup`，純 HTML，無 JS 渲染
- `over18=1` Cookie 繞過成年確認（唯一阻擋）
- 70+ 日韓品牌關鍵字清單（visee、CEZANNE、3CE、rom&nd 等）篩選相關文章

#### Dcard 美妝版
- 全站 Cloudflare Turnstile 人機驗證，純 requests 會 403
- Playwright 啟動真實 Chromium（`headless=False`）通過驗證
- 攔截 `service/api/v2/posts` XHR，直接取 JSON，不解析 HTML

#### @cosme 排名爬蟲（日本）
- `requests` + `BeautifulSoup` 解析 SSR HTML（`dl.clearfix` 結構）
- 嘗試多個 API endpoint 全數 404，最終改爬 HTML 成功
- `deep-translator` 日文→繁中；24 分類共 480 筆

#### 화해（Hwahae）排名爬蟲（韓國）
- 月活躍用戶 1,000 萬+，定位與 @cosme 對稱
- 提供分類商品排名、用戶評分、成分安全評級
- `requests` + JSON Gateway API；`deep-translator` 韓文→繁中
- 與 @cosme 爬蟲輸出同格式 CSV，方便統一處理

**架構對稱設計**：日本 → @cosme（排名 + 評分）；韓國 → 화해（排名 + 評分 + 成分安全評級）

---

*國立雲林科技大學 資訊管理系畢業實務專題・2026*
