# LLM Wiki Schema

這是一個由 LLM 維護的研究知識庫。本文件定義了知識庫的結構、約定和工作流程。

## 語言偏好

**所有回覆與檔案內容一律使用繁體中文。**

- 對話回覆：繁體中文
- Wiki 頁面內容：繁體中文
- 檔案命名：繁體中文或英文皆可
- 程式碼註解（如有）：繁體中文

## 目錄結構

```
├── raw/                 # 原始來源（唯讀，LLM 不修改）
│   └── assets/          # 圖片、PDF 等附件
├── wiki/                # Wiki 頁面（LLM 維護）
│   ├── index.md         # 總索引
│   ├── log.md           # 操作日誌
│   ├── overview.md      # 主題總覽與當前理解
│   ├── sources/         # 來源摘要頁
│   ├── concepts/        # 概念解釋頁
│   ├── entities/        # 實體頁（人物、組織、工具等）
│   └── synthesis/       # 綜合分析、比較、論點
└── CLAUDE.md            # 本文件（Schema）
```

## 頁面約定

### Frontmatter
每個 wiki 頁面應包含 YAML frontmatter：

```yaml
---
title: 頁面標題
type: source | concept | entity | synthesis
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: [相關來源檔名]
tags: [標籤列表]
---
```

### 連結
- 使用 `[[wiki 連結]]` 格式進行頁面間引用
- 首次提及重要概念或實體時，建立連結
- 來源引用格式：`[^source-name]` 或 `[[sources/來源名稱|簡稱]]`

### 命名
- 檔名使用小寫，單詞用連字符連接：`machine-learning.md`
- 中文主題可用中文檔名：`強化學習.md`
- 來源摘要檔名反映原始來源：`sources/attention-is-all-you-need.md`

## 工作流程

### 1. Ingest（攝入來源）

當使用者添加新來源到 `raw/` 時：

1. **閱讀來源**：完整閱讀，理解核心內容
2. **與使用者討論**：總結關鍵發現，確認重點
3. **建立來源摘要**：在 `wiki/sources/` 建立摘要頁
4. **更新相關頁面**：
   - 更新或建立涉及的概念頁 (`concepts/`)
   - 更新或建立涉及的實體頁 (`entities/`)
   - 檢查與現有知識的聯繫和矛盾
5. **更新索引**：在 `wiki/index.md` 添加條目
6. **記錄日誌**：在 `wiki/log.md` 追加操作記錄
7. **更新總覽**：如有重大發現，更新 `wiki/overview.md`

### 2. Query（查詢）

當使用者提問時：

1. **查閱索引**：讀取 `wiki/index.md` 定位相關頁面
2. **深入閱讀**：讀取相關 wiki 頁面
3. **綜合回答**：基於 wiki 內容回答，標註來源
4. **考慮歸檔**：如果答案有複用價值，建議存為 `synthesis/` 頁面

### 3. Lint（維護）

定期或使用者請求時：

- 檢查孤立頁面（無入鏈）
- 檢查失效連結
- 識別矛盾的資訊
- 發現缺失的概念頁
- 建議可以探索的問題
- 建議可以查找的來源

## 頁面模板

### 來源摘要 (sources/)
```markdown
---
title: 來源標題
type: source
created: YYYY-MM-DD
updated: YYYY-MM-DD
original: raw/檔案路徑
tags: []
---

# 來源標題

**作者**：
**日期**：
**類型**：論文 | 文章 | 書籍 | 影片 | 其他

## 核心觀點

## 關鍵內容

## 與其他來源的聯繫

## 問題與思考
```

### 概念頁 (concepts/)
```markdown
---
title: 概念名稱
type: concept
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
tags: []
---

# 概念名稱

## 定義

## 詳細解釋

## 相關概念

## 來源引用
```

### 實體頁 (entities/)
```markdown
---
title: 實體名稱
type: entity
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
tags: []
---

# 實體名稱

## 基本資訊

## 主要貢獻/特點

## 相關實體

## 來源引用
```

### 綜合分析 (synthesis/)
```markdown
---
title: 分析標題
type: synthesis
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
tags: []
---

# 分析標題

## 問題/主題

## 分析

## 結論

## 來源引用
```

## 注意事項

- **保持客觀**：區分事實與觀點，標註不確定性
- **標註來源**：重要論斷必須可追溯
- **發現矛盾**：當新來源與現有知識衝突時，明確記錄
- **漸進更新**：每次攝入後更新相關頁面，保持知識庫一致性
- **使用者主導**：LLM 執行維護工作，但重大決策由使用者確認
