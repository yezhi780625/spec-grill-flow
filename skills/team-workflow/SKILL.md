---
name: team-workflow
description: 團隊 AI 交辦任務流程（Spec-Kit + grill + Superpowers）。開始任何 feature、bug fix 或程式碼變更前先用本流程分流；撰寫或修改 spec、規劃任務、開 PR、驗收、合併後 retro 時也使用。
---

團隊交辦 AI 任務的流程。核心原則：**spec 文件是唯一真相源**——所有需求結論最終寫回 `specs/` 下的 spec 文件，未寫回的結論視為不存在。

> 本檔是濃縮版；與 [full-workflow.md](full-workflow.md) 不一致時，以 full-workflow.md 為準。

```
分流 ─┬─ 完整通道：定義 → 錘鍊 → 規劃 → 實作 → 驗收 → Retro
      ├─ 輕量通道：寫入即錘鍊 ──→ 簡化規劃 → 實作 → 驗收 → Retro
      └─ 快速通道：──────────────────────→ 實作 → 驗收 → Retro
```

**運作模式**（Phase 0 已宣告，見 constitution Governance）：Team 模式的「非本人」角色由真人擔任；Solo 模式由 **fresh-context 獨立 subagent** 代位——可自由讀取 repo，但不共享產生受審產物的對話脈絡，prompt 指定挑錯立場。

## 分流（每個任務的第一步）

依**任務性質**判斷，不需先讀完 spec 預測差異。猶豫時一律選更完整的通道：

- **快速通道**：任務是 typo／依賴升級／bug fix（回到 spec 已描述的行為）／不改外部行為的重構，且不引入新依賴、不新增抽象層 → 直接進 Phase 4。「約 3 個檔案」是絆線非門檻：事前免精算，PR diff 超過 3 檔時由 reviewer 判斷是否升級。
- **輕量通道**：只新增或修改 **1 條**驗收標準，不引入新依賴、不新增抽象層 → 直接在 spec 增修該條並 commit，立刻只針對它跑 grill 四維度各至少一問（執行者規則同 Phase 2），tasks 補一條含測試任務的描述（免完整 plan），進 Phase 4。
- **完整通道**：其餘一切（新 feature、動多條標準、新依賴／抽象層、拿不準）→ Phase 1 起走完。

**升級規則（單向）**：進行中發現超出通道資格——立即停手，升級（快速→輕量或完整；輕量→完整）。**已寫的測試與程式碼不作廢**，帶入 Phase 4 繼續，只回頭補 spec 與規劃。猶豫一律升級，不得反向降級。

## Phase 1：定義

用 `/speckit.specify` 把原始需求產成 spec 草稿，commit 進 feature branch。

**完成條件**：spec 草稿已 commit。此時 spec 未經檢驗，直接進 Phase 3 是流程違規。

## Phase 2：錘鍊（grill）

**執行者不得是 spec 作者**（team：他人主持；solo：fresh-context subagent，不帶 Phase 1 對話脈絡、repo 可查、adversarial 立場）。用 `grill-me` 對著 spec 逼問，每一輪修正**直接改 spec 並 commit**——git log 就是訪談紀錄。追問至少涵蓋：

- 每條驗收標準怎麼量測？由誰／什麼工具判定通過？
- Edge cases 與失敗路徑列了嗎？
- 需求之間有無矛盾？
- 「不做什麼」明確了嗎？

grill 的產出只寫回 spec 本身（不產 ADR、glossary 或其他文件）。作者可駁回追問，但被駁回的追問由 grill 執行者記入 spec「未解事項」段——不得刪除，Phase 5 reviewer 會檢視。

**完成條件**：spec 頂部的 grill 檢核清單四項全勾：

```markdown
## Grill 檢核
- [ ] 所有驗收標準可量測
- [ ] Edge cases 與失敗路徑已列出
- [ ] 無互相矛盾的需求
- [ ] 「不做什麼」已明確
```

## Phase 3：規劃

`/speckit.plan` 產技術計畫 → `/speckit.tasks` 拆任務，一併 commit。過程中發現 spec 有洞 → 退回 Phase 2 補 grill 後再回來（計入退回次數）。

**完成條件**：每條驗收標準在 tasks 中有對應的測試任務。

## Phase 4：實作

依 tasks 逐項實作，走 Superpowers 紀律：

- TDD 強制：測試先寫、先失敗，才准實作。豁免類別（純 UI 樣式／一次性 migration／無邏輯膠水碼）走 constitution I 豁免程序：PR 明寫理由＋替代驗證方式，禁止空測試充數。
- 除錯：先找根因，才准修。
- 實作中發現需求問題 → 停手，退回 Phase 2 改 spec、過 grill，再繼續（計入退回次數）。需求變更一律經過 spec——spec 與程式碼不同步是本流程最大的失效模式。

**完成條件**：所有 tasks 完成且測試綠燈。

## Phase 5：驗收

開 PR，reviewer（非實作者本人；solo 模式由 fresh-context agent 代位，可自由讀取 repo、僅不帶實作過程的對話脈絡）將 spec 驗收標準貼入 PR 逐條驗收：

```markdown
## Spec 驗收（對照 specs/xxx/spec.md）
- [ ] 每條驗收標準有證據（自動測試連結，或人工驗證步驟與結果）
- [ ] 所有 edge cases 有對應測試
- [ ] 未實作 spec 範圍外的功能（scope creep 檢查）
- [ ] spec「未解事項」各項已定案——標「已接受取捨（凍結）」或轉 issue 移出 spec
```

任一條不過 → 打回 Phase 4；根因在 spec → 退回 Phase 2。全過 → 合併（合併前「未解事項」每項須定案：凍結或轉 issue，不得無狀態滾入下一次修改）。抽查測試品質：revert 對應實作後測試必須轉紅，空測試視同無測試。

- 快速通道：免逐條對照，確認「失敗測試→修正→綠燈」證據鏈；diff 超過 3 檔時判斷是否應升級通道。
- 輕量通道：逐條對照僅限本次增修的標準與其 edge cases。

**完成條件**：驗收清單全勾且已合併。

## 退回與熔斷

- 同一 feature **累計退回 Phase 2 達 2 次** → 熔斷：停止流程，**強制拆分**——已想清楚的部分保留成獨立 spec 續行，只把糾纏不清的部分回 Phase 1 重新定義（team：拆分決策拉高為專案級討論；solo：換 fresh agent 從問題定義主導拆分）。retro 症狀記「熔斷」。
- **Phase 5→4 打回達 2 次** → 第三次實作前強制先做根因分析並寫入 PR。

## Phase 6：Retro（合併後，2 分鐘內完成）

合併後立即在 `specs/retro-log.md` 回填一行（欄位：合併日、feature、通道、開工日、退回次數＝P3→P2＋P4→P2、打回次數＝P5→P4＋P5→P2、觸發條款＝本次真正攔下問題或改變決定的條款（絆線／豁免／熔斷／未解事項／grill 修正，無則留空）、症狀分類、一句話備註）。症狀分類使用下方失效模式表的既有分類；新型症狀在失效模式表新增一列並在備註標注「新增」。

- 快速／輕量通道與零退回零打回的任務同樣回填——前者偵測分流條件是否太寬，後者是打回率的分母。
- 檢視雙保險：**每累積 10 筆或滿一季**（先到者），由回填觸發筆的人當週執行一次檢視——繞過偵測（merged feature PR 數 − retro-log 新增列數，分子排除自動化／docs-only／revert／CI 修正）、條款有效性（以「觸發條款」欄為準，連續兩次零觸發 → 刪除候選）、分流條件鬆緊、cycle time 是否惡化、grill-me 上游同步、附錄 B 條件。

**完成條件**：retro-log 已回填；若剛好是第 10n 筆，檢視已排入當週。

## 工具邊界（reference）

| 動作 | 用 | 附註 |
|---|---|---|
| spec / plan / tasks | `/speckit.*` | Superpowers 的 /brainstorm、/write-plan 不在本流程使用 |
| 錘鍊需求 | `grill-me` 對 spec 文件 | 產出只寫回 spec |
| 實作、TDD、除錯 | Superpowers | — |
| 品質門檻 | `.specify/memory/constitution.md` | speckit 各階段自動引用 |

## 失效模式（reference）

| 症狀 | 對策 |
|---|---|
| 做出來的不是想要的 | 檢查 grill 執行者是否獨立於 spec 作者（非作者主持／fresh-context agent） |
| 回報完成但驗收失敗 | Phase 3 起驗收標準就要對應測試任務 |
| spec 和程式碼對不上 | 執行「改需求必退 Phase 2」 |
| 同一 feature 反覆退回繞圈 | 觸發熔斷：強制拆分，糾纏部分回 Phase 1 重新定義 |
| 測試存在但形同虛設 | 抽查「revert 實作測試須轉紅」；適用豁免就走豁免程序，不寫假測試 |
| 成員產出品質落差 | PR review 檢查 spec 的 grill 檢核與 commit 歷史 |
| 快速／輕量通道打回率升高 | 分流條件太寬，收緊任務類型白名單 |
| 打回率漂亮但交付變慢 | 檢視 cycle time（開工日→合併日），放寬非關鍵關卡 |
| 流程被私下繞過（retro-log 看不見） | 檢視時比對 merged PR 數與 retro-log 列數，差值擴大 → 優先刪流程 |
| 測試都過但仍有安全／架構問題 | 缺獨立驗收——引入 ECC reviewer，條件見 [full-workflow.md](full-workflow.md) 附錄 B |

## 完整版文件

各階段負責人與代位規則、Phase 0 專案初始化、熔斷細節、ECC reviewer 引入條件（附錄 B）見 [full-workflow.md](full-workflow.md)。
