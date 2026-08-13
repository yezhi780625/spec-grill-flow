# 專案 Constitution

> **用法**：把本檔放到 `.specify/memory/constitution.md` 直接使用，或作為 `/speckit.constitution` 的輸入讓 agent 填合專案脈絡。
> 每條原則採「Rule（規則）→ Rationale（理由）→ Verification（如何判定違反）」三段式。
> Rule 寫給 agent 執行，Verification 寫給 reviewer 判定——**寫不出 Verification 的原則就是空話，不要收錄**。
> `<角括號>` 為需依專案填寫的參數。

---

## Core Principles

### I. Test-First（NON-NEGOTIABLE）

**Rule**：任何實作程式碼提交前，對應測試必須已存在且經歷過失敗狀態（red-green-refactor）。
**豁免**：純 UI 樣式調整、一次性 migration script、無邏輯的設定與膠水碼可豁免 test-first，但 PR 必須明寫豁免理由與**替代驗證方式**（人工驗證步驟、截圖、migration dry-run 輸出），由 reviewer 覆核豁免正當性。禁止以空測試或恆真斷言充數——那會污染驗收證據鏈，比沒有測試更糟，視同違反本條。
**Rationale**：先失敗的測試是驗收標準被真正執行的唯一證據；後補的測試只是為既有行為背書。無豁免條款的鐵律只會誘發應付檢查的假測試。
**Verification**：PR 中測試的 commit 早於或同於實作 commit；抽查任一測試，revert 對應實作後測試必須轉紅。豁免的 PR 須有理由與替代驗證方式，缺一即違反。

### II. 驗收標準可量測（NON-NEGOTIABLE）

**Rule**：spec 中每條驗收標準必須指明判定方式——自動測試優先；無法自動化者，寫明人工驗證步驟與預期結果。
**Rationale**：無法量測的標準等於把驗收交給各人心證，AI 回報「完成」時無從反駁。
**Verification**：spec 中任一條驗收標準若無法回答「由誰／什麼工具、怎麼判定通過」，即違反。

### III. Grill Gate

**Rule**：spec 通過 grill 檢核四項（驗收標準可量測／edge cases 與失敗路徑已列／無矛盾需求／「不做什麼」已明確）並全數勾選後，才得執行 `/speckit.plan`。
**Rationale**：模板只保證格式完整，不保證想法正確；grill 是 spec 品質的唯一壓力測試。
**Verification**：spec 文件頂部檢核清單四項全勾，且 git log 顯示 grill 修正的 commit 歷史。無修正紀錄的「一次過」spec 應被質疑。

### IV. Spec 為唯一真相源（NON-NEGOTIABLE）

**Rule**：需求變更一律先修改 spec 文件、重新通過 grill 檢核，再繼續實作。
**Rationale**：spec 與程式碼不同步時，驗收失去依據，整條流程的驗證能力歸零。
**Verification**：PR 的程式碼行為若與 spec 描述不符（多做、少做、或不同做法），即違反；「程式碼是對的，spec 忘了改」同樣算違反。

### V. 簡潔優先（YAGNI）

**Rule**：採用能滿足 spec 的最簡方案。新增抽象層、服務拆分、或引入新依賴，必須在 plan 中附書面理由，說明「現在的需求」（而非未來想像）為何需要它。
**Rationale**：AI agent 天然傾向過度工程；沒有明文煞車，複雜度只會單向增長。
**Verification**：plan 中出現未附理由的新抽象／新服務／新依賴，即違反。理由訴諸「未來可能需要」者視同未附理由。

### VI. 根因除錯

**Rule**：修 bug 前必須先定位並在 PR 或 commit message 中寫明根因；修正必須針對根因，並附防止回歸的測試。
**Rationale**：症狀式修補讓同一類 bug 反覆出現，且污染程式碼。
**Verification**：bug fix 的 PR 若無根因說明或無回歸測試，即違反。

### VII. <專案特定原則，例：安全>

**Rule**：<例：所有資料庫查詢使用參數化語句；所有使用者輸入在信任邊界驗證；禁止自行實作加密。>
**Rationale**：<為什麼這對本專案不可協商。>
**Verification**：<reviewer 或掃描工具如何判定違反。>

<!-- 依專案增補：效能門檻（p95 < Xms）、相依政策（優先成熟函式庫）、可觀測性（結構化日誌）等。
     每加一條先通過測試：想像不出「agent 違反它的具體場景」就不要加。 -->

---

## Workflow Gates

流程各階段的放行條件（與 team-workflow skill 對應）：

| Gate | 放行條件 | 依據原則 |
|---|---|---|
| specify → plan | grill 檢核四項全勾，且 grill 由非作者（team）或 fresh-context agent（solo）執行 | III |
| plan → tasks | 新增複雜度皆附書面理由 | V |
| tasks → implement | 每條驗收標準有對應測試任務 | I, II |
| implement → merge | 驗收清單全勾、由非實作者 review | II, IV |

## Governance

- **運作模式**：<Solo（個人＋AI）或 Team>。Solo 模式下所有「非作者／非實作者」角色由 fresh-context 獨立 AI agent 代位（代位規則見 team-workflow skill）。
- **Owner**（Phase 0 填齊，不留空）：constitution 修訂核准人 <姓名>；grill-me 上游同步 <姓名>；ECC agent（若引入）<姓名>。Solo 模式三者皆為本人，職責由 retro 檢視節奏觸發。
- 本 constitution 效力高於其他慣例文件；與 CLAUDE.md 或 skill 衝突時，以本文件為準。
- 修訂須經 PR review，由上列核准人核准；修訂需附動機（通常是 PR 驗收中反覆出現的同類問題）。
- 標註 NON-NEGOTIABLE 的原則，修訂需 <更高門檻，例：全隊同意>。
- 檢視節奏採數據量驅動：retro-log 每累積 10 筆檢視一次（取代日曆制季檢，觸發規則見 team-workflow Phase 6）。連續兩次檢視未攔下任何違反的原則，評估是否已成空話、應精煉或移除。

**Version**: 0.1.0 | **Ratified**: <日期> | **Last Amended**: <日期>
