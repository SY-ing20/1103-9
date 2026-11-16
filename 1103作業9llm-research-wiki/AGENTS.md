# AGENTS.md — LLM 研究知識庫維護規範

本文件為 LLM Agent 維護此知識庫（Wiki）的執行規範。此知識庫採用「持續累積編譯（Compound Wiki）」架構：原始資料保持不變，所有摘要、概念解析、模型檔案、基準測試比較與研究綜述皆由 Agent 負責撰寫、更新與維護交叉連結。

---

## 1. 目錄架構與分層定義

知識庫分為三個核心層級：

### 第一層：原始資料層（`raw/`）— 唯讀
- `raw/papers/`：論文 PDF 或透過工具轉換的 Markdown 文本。
- `raw/articles/`：技術部落格、研究筆記、會議紀錄（建議使用 Obsidian Web Clipper 擷取）。
- `raw/assets/`：原始文獻中的架構圖、實驗數據圖表與本地附件。
- **操作限制**：Agent 僅可讀取 `raw/` 內容，嚴禁修改、重新命名或刪除其中的任何檔案。

### 第二層：知識庫層（`wiki/`）— 由 Agent 全權維護
- `wiki/index.md`：全庫內容目錄（按分類條列所有頁面、一句話摘要與更新日期）。
- `wiki/log.md`：按時間順序追加的操作日誌（僅可新增，不可覆寫歷史紀錄）。
- `wiki/overview.md`：LLM 研究領域的總體知識地圖與核心研究主線。
- `wiki/sources/`：單篇文獻的結構化摘要頁（一對一對應 `raw/` 中的資料）。
- `wiki/concepts/`：核心技術概念頁（如 Attention 變體、RLHF、GRPO、Scaling Laws、KV Cache 壓縮）。
- `wiki/models/`：模型與架構專頁（如 DeepSeek-R1、Llama-3、Qwen-2.5、Mamba）。
- `wiki/benchmarks/`：評測基準與數據集頁（如 MMLU-Pro、SWE-bench、GPQA、AIME）。
- `wiki/syntheses/`：跨文獻比較表、技術演進分析與問答沉澱報告。
- `wiki/templates/`：各類頁面的標準 Markdown 模板。

### 第三層：規範與部署層
- `AGENTS.md`：本規範文件。
- `mkdocs.yml` 與 `.github/workflows/deploy.yml`：靜態網頁建置與自動部署設定檔。

---

## 2. 頁面格式與寫作規範

### 2.1 YAML Frontmatter 標頭
所有 `wiki/` 下的內容頁面（`index.md` 與 `log.md` 除外）必須包含以下 YAML 標頭：

```yaml
---
title: 頁面標題
type: source | concept | model | benchmark | synthesis
tags: [領域標籤1, 領域標籤2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources_count: 1
status: draft | active | superseded
---
```

### 2.2 連結與引用規則
- **內部雙向連結**：引用庫內其他頁面時，一律使用標準 Wikilink 格式 `[[頁面檔名]]` 或 `[[頁面檔名|顯示名稱]]`（檔名使用小寫英文與連字號，例如 `[[group-relative-policy-optimization]]`）。
- **來源溯源**：所有數據、實驗結論與論點必須明確標註出處，連結至對應的 `wiki/sources/` 頁面。
- **矛盾標記**：若新文獻的實驗結果推翻或修正既有頁面的結論，不得直接刪除舊有敘述，應在該段落加入 `> [!WARNING] 觀點更新` 區塊，並列明新舊來源與差異原因。

### 2.3 行文風格
- 採用客觀、平淡、精確的技術寫作語氣，避免情緒化詞彙與修飾性形容詞。
- 優先使用結構化條列、數據對照表與數學公式（LaTeX 格式）呈現核心機制與實驗結果。

---

## 3. 核心工作流程（Operations）

### 3.1 匯入新文獻（Ingest）
當使用者提供新文獻（放置於 `raw/` 或直接貼上內容）並要求匯入時，依序執行以下步驟：
1. **讀取與解析**：完整閱讀 `raw/` 中的目標文件；若包含本地圖片（`raw/assets/`），依序檢視圖表以獲取架構細節與實驗數據。
2. **建立來源摘要頁**：依據 `wiki/templates/paper-summary.md` 在 `wiki/sources/` 建立新頁面（命名格式：`YYYY-MM-DD-簡短英文標題.md` 或 `arxiv-編號.md`）。
3. **更新概念與模型頁**：檢查該文獻涉及的核心技術、模型或基準測試。若 `wiki/concepts/`、`wiki/models/` 或 `wiki/benchmarks/` 中已有對應頁面，將新發現整合進去並更新 `updated` 日期與 `sources_count`；若尚無頁面且該概念具備獨立研究價值，則建立新頁面。
4. **建立交叉連結**：確保新頁面與所有相關既有頁面之間建立雙向 `[[wikilink]]`。
5. **更新目錄與日誌**：
   - 在 `wiki/index.md` 對應分類下新增或更新條目。
   - 在 `wiki/log.md` 檔案最末端追加一筆紀錄。
6. **回報摘要**：向使用者列出本次新增與修改的所有檔案清單，並摘要該文獻對現有知識體系的補充或修正。

### 3.2 查詢與分析（Query）
當使用者針對研究問題提問時：
1. **檢索定位**：優先讀取 `wiki/index.md` 與 `wiki/overview.md`，鎖定相關的 `concepts/`、`models/` 與 `sources/` 頁面。
2. **深度合成**：讀取目標頁面內容，綜合多篇文獻的觀點與數據回答問題，並附上內部頁面引用。
3. **知識沉澱**：若該次回答產出了有價值的跨模型比較表、技術路線分析或問題排查結論，主動詢問使用者是否將此回答歸檔至 `wiki/syntheses/` 並更新 `index.md` 與 `log.md`。

### 3.3 健康檢查（Lint）
當使用者要求執行知識庫檢查（Lint）時，全面掃描 `wiki/` 目錄並產出檢查報告：
1. **矛盾檢測**：檢查不同頁面間對同一技術或基準分數的描述是否存在未標記的衝突。
2. **孤島頁面（Orphan Pages）**：找出沒有任何連入連結（Inbound Links）的頁面，補充相關脈絡連結。
3. **缺失頁面**：找出在多篇文獻中被頻繁以 `[[wikilink]]` 提及但尚未建立實體檔案的概念或模型。
4. **過時資訊**：檢視標記為早期結論的頁面，確認是否有後續論文已提出更優方法或推翻假設。
5. **研究缺口建議**：根據現有知識圖譜的分布，提出 3 至 5 個值得進一步檢索文獻的研究問題。

---

## 4. `log.md` 紀錄格式規範

每次執行 Ingest、Query 歸檔或 Lint 後，必須在 `wiki/log.md` 末尾新增以下格式的標題與簡述（便於使用 `grep "^## \[" wiki/log.md` 檢索）：

```markdown
## [YYYY-MM-DD] <操作類型: ingest | synthesize | lint> | <簡短標題>
- **操作對象**：`wiki/sources/xxx.md`
- **連動更新**：`wiki/concepts/aaa.md`, `wiki/models/bbb.md`
- **摘要**：一句話說明本次變更的核心內容。
```
