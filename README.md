# 日韓藥妝智慧推薦系統 / Buy託了AI

整合日韓商品資料與多平台社群評論，透過情緒分析與 RAG 向量搜尋，依照膚質、預算、需求精準推薦日韓藥妝。

## 資料來源

**商品資料**
- 日本：Rakuten Ichiba API
- 韓國：Olive Young 爬蟲

**評論語料**
- PTT MakeUp 板（requests + BeautifulSoup）
- Dcard 美妝版（Playwright + Cloudflare Turnstile 繞過）
- @cosme 排名爬蟲（日本，SSR HTML 解析）
- 화해（Hwahae）排名爬蟲（韓國）

## 技術架構

- 後端：FastAPI + MongoDB Atlas
- 向量搜尋：ChromaDB + sentence-transformers
- 情緒分析：HuggingFace 中文模型
- 前端：HTML / CSS / JavaScript
- 部署：Render.com（後端）+ GitHub Pages（前端）

## 企劃書

- [線上瀏覽](https://jayee123.github.io/PRJ_HTML_PLAN/drugstore_plan.html)
