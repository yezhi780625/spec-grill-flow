---
name: team-workflow
description: 團隊 AI 交辦任務流程（Spec-Kit + grill + fresh-context subagent 實作）。開始任何 feature、bug fix 或程式碼變更前先用本流程分流；撰寫或修改 spec、規劃任務、實作、開 PR、驗收時也使用。
---

團隊交辦 AI 任務的流程。核心原則：**spec 文件是唯一真相源**——所有需求結論最終寫回 `specs/` 下的 spec 文件，未寫回的結論視為不存在。

> 本檔是濃縮版；與 [full-workflow.md](full-workflow.md) 不一致時，以 full-workflow.md 為準。

```
提案（feature request／PRD／自發 feature）── steelman（挖目標→確認）──確認──┐ （撤回／擋下：不進流程）
回報與無新行為任務（bug fix／typo／重構…）───────────────────────────────┤
                                                                         ▼
                                     分流 ─┬─ 完整通道：定義 → 錘鍊 → 規劃 → 實作 → 驗收
                                           ├─ 輕量通道：寫入即錘鍊 ──→ 簡化規劃 → 實作 → 驗收
                                           └─ 快速通道：──────────────────────→ 實作 → 驗收
```

**人力與 grill 執行者**（Phase 0 宣告專案有無真人；用不用人由分流決定）：完整通道且有真人 → 真人 grill（非作者主持，兼知識擴散——同事從此知道這個 feature 為什麼存在，AI 無法替代）；其餘 → **fresh-context 獨立 subagent** 代位——可自由讀取 repo，但不共享產生受審產物的對話脈絡，prompt 指定挑錯立場。通道升級時 grill 執行者自動升級。

**模型與 effort 不由本流程指定**：主對話的模型與 effort 由使用者以 `/model`、`/effort` 選擇，個人偏好寫在 `.claude/settings.local.json`。派出 subagent（代位 grill、Phase 4 實作、代位驗收）時，model 與 effort 由主 agent 依任務判斷決定；不指定時沿用主對話的設定。

## 目標確認（分流之前，僅限提案）

只攔**提案**（要求新行為：feature request、PRD、方案性需求、內部自發 feature），不攔**回報**。判別式：對方期望的行為，spec（或現有明確行為）已描述了嗎？已描述 → bug 回報，目標自明，免跑直接分流（即快速通道的 bug fix）；未描述 → 是包裝成 bug 的需求變更，照跑。拿不準 → 跑（成本只是一則回覆）。內部無新行為任務（typo、重構等）同樣免跑。

提案先跑 `steelman-requirement`：挖出需求背後的目標、重建最強版本、給意見、以確認句核對目標（由 RD 向 PM 代為轉達——是轉達不是代拍板）。本站不裁決「該不該做」（那是產品決策，RD 無否決權），保證的是**目標確認完成才寫 spec**：目標已確認 → 進分流，「需求背後的目標」段落隨需求原文帶入 Phase 1 作 spec 動機雛形，未被採納的疑慮以「已知取捨」寫入 spec 未解事項；目標未確認 → 不寫 spec。PM 經 steelman 後撤回或改寫需求，或內部提案被擋下（內部自發 feature 保留擋下語義——RD 有裁量權；其 steelman 執行者不得是提案者本人：同事或 fresh-context subagent 代位）→ 不進流程，改寫後的新需求重新過一次目標確認。steelman 確立「為什麼做」，不取代 Phase 2 grill 的「怎麼做才驗收得了」——不可互抵。

## 分流（每個任務的第一步；提案先過目標確認）

依**任務性質**判斷，不需先讀完 spec 預測差異。猶豫時一律選更完整的通道：

- **快速通道**：任務是 typo／依賴升級／bug fix（回到 spec 已描述的行為）／不改外部行為的重構，且不引入新依賴、不新增抽象層 → 直接進 Phase 4。「約 3 個檔案」是絆線非門檻：事前免精算，PR diff 超過 3 檔時由 reviewer 判斷是否升級。
- **輕量通道**：只新增或修改 **1 條**驗收標準，不引入新依賴、不新增抽象層 → 直接在 spec 增修該條並 commit，立刻只針對它跑 grill 四維度各至少一問（AI 代位執行），tasks 補一條含測試任務的描述（免完整 plan），進 Phase 4。
- **完整通道**：其餘一切（新 feature、動多條標準、新依賴／抽象層、拿不準）→ Phase 1 起走完。

**升級規則（單向）**：進行中發現超出通道資格——立即停手，升級（快速→輕量或完整；輕量→完整）。**已寫的測試與程式碼不作廢**，帶入 Phase 4 繼續，只回頭補 spec 與規劃。猶豫一律升級，不得反向降級。

**分流後第一步：重設 `.specify/feature.json`**：spec-kit 1.x 只靠這個檔決定 speckit 指令作用在哪個 feature（本機檔案，不進版控），不看 git 分支；它不會自己清空，留著舊值時指令會默默作用在上一個 feature 上。所以每次開始一項工作就重設它，不沿用現有的內容：

- 完整通道：`/speckit-specify` 會寫入新的 feature 目錄，不必自己動。
- 輕量通道：寫成 `{"feature_directory":"specs/<被修改的 spec 目錄>"}`。
- 快速通道：刪除它。快速通道不動 spec、不跑 speckit 指令；檔案不在時，誤跑的指令會直接報錯。
- 接手既有的工作（切換分支、新建的 worktree、reviewer 在自己的 checkout 重跑）：寫成這次要動的 spec 目錄。新建的 worktree 裡沒有這個檔，派 subagent 過去時把這一步寫進交辦內容。
- 通道升級後：照新通道的做法再做一次。

路徑用相對於 repo 根目錄的寫法，以這次要動的 spec 所在目錄為準，不從分支名推導（一個 feature 常拆成多個分支）。

## Phase 1：定義

用 `/speckit-specify` 把原始需求產成 spec 草稿，接著建立 feature branch（`git switch -c <分支名> <基準分支>`；基準分支是預設分支或這條工作線的整合分支；命名依專案慣例，沒有慣例時用 feature 目錄名，例如 `001-duration-parsing`；已在這次工作的分支上則跳過），把草稿 commit 進去。專案保留了 spec-kit 的 git extension 時例外：它的 hook 會在 `/speckit-specify` 之前從當下的分支開出 `<編號>-<名稱>` 分支，所以先切到基準分支再執行 `/speckit-specify`，這一步跳過。經目標確認的任務，把 steelman 定案的「需求背後的目標」段落連同需求原文一起輸入，作為 spec 動機雛形。

**完成條件**：spec 草稿已 commit。此時 spec 未經檢驗，直接進 Phase 3 是流程違規。

## Phase 2：錘鍊（grill）

**執行者不得是 spec 作者**——完整通道且有真人：非作者成員主持；其餘：fresh-context subagent（不帶 Phase 1 對話脈絡、repo 可查、adversarial 立場；它無法直接向作者提問，每輪問題由主對話轉給作者，作答後交回同一個 subagent 續問；主對話只能原文轉送，不得改寫、加註或代答；每輪由 subagent commit，主旨以 `docs: grill` 開頭、內文含問題與作者答案原文，主旨與內文每行都不超過 100 字元（超過就斷行，不改寫），沒改 spec 的輪次用空 commit）。用 `grill-me` 對著 spec 逼問，每一輪修正**直接改 spec 並 commit**——git log 就是訪談紀錄。追問至少涵蓋：

- 每條驗收標準怎麼量測？由誰／什麼工具判定通過？
- Edge cases 與失敗路徑列了嗎？
- 需求之間有無矛盾？
- 「不做什麼」明確了嗎？

grill 的產出只寫回 spec 本身（不產 ADR、glossary 或其他文件）。作者可駁回追問，但被駁回的追問由 grill 執行者記入 spec「未解事項」段——不得刪除，Phase 5 reviewer 會檢視。真人主持時可主動補充作者不掌握的事實（寫入 spec 標注來源，拍板仍屬作者）；一次 grill 出現 ≥3 項此類補充 → 開 issue 把這些知識補進專案文件；補上之前，該領域的任務分流偏向完整通道。

**完成條件**：spec 頂部的 grill 檢核清單四項全勾：

```markdown
## Grill 檢核
- [ ] 所有驗收標準可量測
- [ ] Edge cases 與失敗路徑已列出
- [ ] 無互相矛盾的需求
- [ ] 「不做什麼」已明確
```

## Phase 3：規劃

`/speckit-plan` 產技術計畫 → `/speckit-tasks` 拆任務 → `/speckit-analyze` 檢查 spec、plan、tasks 與 constitution 是否一致（唯讀；僅完整通道，輕量通道免完整 plan，不適用），一併 commit。analyze 的 CRITICAL 與 HIGH 發現要回源頭修正後重跑：問題在 plan 或 tasks 就在本階段修；問題在 spec，或過程中發現 spec 有洞 → 退回 Phase 2 補 grill 後再回來（計入退回次數）；不同意的發現附理由記入 spec「未解事項」，不必為它重跑。結果寫進 plan 與 tasks 那個 commit 的內文（例如 `analyze: 0 CRITICAL, 0 HIGH`），不當主旨的前綴。

`/speckit-tasks` 只在 spec 或 constitution 要求時才產生測試任務。專案的 constitution 沒有等效於 Test-First 的條文時，呼叫 `/speckit-tasks` 要明確要求 TDD。

**完成條件**：每條驗收標準在 tasks 中有對應的測試任務；完整通道另須 `/speckit-analyze` 沒有未處理的 CRITICAL 或 HIGH 發現（已修正，或已記入未解事項）。

## Phase 4：實作

主 agent 依 tasks.md 的順序，把任務交給 **fresh-context subagent** 實作；主 agent 負責派工與核對回報。主 agent 不寫測試與實作，小修改也一樣派 subagent（使用者明確要求直接改時除外）；整合時頂多解合併衝突。主 agent 載入 `/speckit-implement` 當入口也可以，派工、commit 與勾選仍依下列規則。

**派工**：

- 一次交辦的範圍由主 agent 決定（單一任務，或 tasks.md 的一個階段／一個 user story）。model 與 effort 由主 agent 判斷。
- 可以並行派工：彼此不依賴的任務可以同時交給多個 subagent。**一個 agent 一個 git worktree**（派工工具的隔離選項或 `git worktree add` 都可以）——共用工作目錄會互相覆蓋檔案、打亂 commit。同時幾個、成果走本機合併還是各自開 PR、由誰解衝突，依專案的做法。
- 整合時保留原本的 commit（merge，不 squash——Phase 5 靠 commit 驗證測試先行），整合後執行完整測試。
- 交辦內容用路徑指向 spec、plan、tasks.md 與 constitution，列出這次的任務編號，並附上下方「實作規則」全文——subagent 看不到主對話，也不會自己載入本 skill。
- subagent 回報後，先確認完整測試綠燈（自己執行，或看 CI 的結果），才接受這批成果。

**實作規則**（給 subagent）：

- TDD（constitution 的 Test-First 原則）：每個行為先寫測試，執行並看到它失敗，才寫實作。證據有兩種，各管各的範圍：
  - 新行為：測試的 commit 排在對應實作的 commit 之前，commit 訊息帶任務編號。commit 訊息照專案的 commit 慣例寫（有 commitlint 就要過得了）；任務編號放主旨結尾的括號裡（例如 `test: <描述> (T012)`），不放開頭。
  - 事後新增或修改的斷言（補強測試、文案或錯誤碼改動後的斷言）：逐個證明它會失敗——把對應的行為改壞、確認該斷言轉紅、再還原——並把改壞了什麼、哪個斷言轉紅寫進 PR 描述。「不包含某字串」這類負向斷言尤其要做：文案一改，它可能永遠成立。

  豁免類別（純 UI 樣式／一次性 migration／無邏輯膠水碼）走 Test-First 原則的豁免程序：PR 明寫理由＋替代驗證方式，禁止空測試充數。
- 勾選：這個 PR 完成的任務，在這個 PR 裡把 tasks.md 對應的 `- [ ]` 改成 `- [x]` 並 commit。
- 開 PR（或 MR）時，以專案的 PR 範本作為描述的骨架，逐段填寫。
- 除錯（constitution 的「根因除錯」原則）：修 bug 一律先寫出重現問題的失敗測試，針對根因修正，根因寫進 commit 訊息。原因還不清楚時（測試以非預期的方式失敗、既有測試轉紅、問題時有時無、或第一次修正後紅燈沒有轉綠），先載入 `diagnosing-bugs` skill 再動手。
- 發現需求問題 → 停手並回報。由主 agent 退回 Phase 2 改 spec、過 grill，再繼續（計入退回次數）。需求變更一律經過 spec——spec 與程式碼不同步是本流程最大的失效模式。

快速通道沒有 tasks.md：交辦內容就是該次任務本身（bug fix 先寫重現問題的失敗測試），勾選不適用。

**開 PR 前**（完整通道）跑 `/speckit-converge`：它對照 spec、plan、tasks 檢查程式碼，不改程式碼，只把剩餘工作附加到 tasks.md。附加了任務 → 照上述規則完成後重跑，直到回報 `Converged`，並把結果寫進 PR 描述。這是自我檢查，不取代 Phase 5，也不計入打回次數。輕量通道可選用（feature 目錄已有 plan.md 與 tasks.md 時，以參數限定範圍）。

**完成條件**：完整測試綠燈；完成的任務都已在 tasks.md 勾選（快速通道沒有 tasks.md，免）；新行為的測試 commit 在實作之前，事後新增或修改的斷言有失敗證據（豁免者除外）；完整通道另須 `/speckit-converge` 回報 `Converged`。

## Phase 5：驗收

開 PR，reviewer（非實作者本人；無真人可用時由 fresh-context agent 代位，可自由讀取 repo、僅不帶實作過程的對話脈絡）將 spec 驗收標準貼入 PR 逐條驗收：

```markdown
## Spec 驗收（對照 specs/xxx/spec.md）
- [ ] 每條驗收標準有證據（自動測試連結，或人工驗證步驟與結果）
- [ ] 所有 edge cases 有對應測試
- [ ] 測試先行有證據：新行為的測試 commit 早於實作 commit；事後新增或修改的斷言附失敗證據（豁免者見 PR 的豁免段）
- [ ] tasks.md 的勾選與本 PR 完成的任務一致
- [ ] 未實作 spec 範圍外的功能（scope creep 檢查）
- [ ] `/speckit-converge` 結果為 Converged（完整通道，結果附在 PR 描述）
- [ ] spec「未解事項」各項已定案——標「已接受取捨（凍結）」或轉 issue 移出 spec
- [ ] 代位 grill（AI 執行時）：本分支有 `docs: grill` 開頭的 commit（0.7.2 之前寫下的 `grill:` 開頭也算），且每個都含該輪問題與作者答案原文；有 `docs: grill [補問]` commit 時只檢查補問的 commit
```

任一條不過 → 打回 Phase 4；根因在 spec → 退回 Phase 2；代位 grill 問答紀錄不符（本分支沒有 `docs: grill` 開頭的 commit——0.7.2 之前的 `grill:` 開頭也算——或有 commit 缺問答原文）→ 由新的 fresh-context subagent 重跑一輪 grill 四維度，commit 主旨以 `docs: grill [補問]` 開頭、內文含問答原文，之後只檢查補問的 commit；只補紀錄不計入退回／打回與熔斷，導致 spec 修改則計為 P5→P2。全過 → 合併（合併前「未解事項」每項須定案：凍結或轉 issue，不得無狀態滾入下一次修改）。確認測試先行：抽查時簽出新行為的測試 commit，測試應為紅燈。抽查測試品質以斷言為單位：挑本 PR 新增或修改的斷言，改壞它對應的行為，該斷言必須轉紅；永遠成立的斷言視同沒有測試。

- 完整通道的 converge 那一勾：PR 描述沒有 `Converged` 的結果，或 reviewer 重跑 `/speckit-converge` 後附加了任務 → 打回 Phase 4，照常計入打回次數。
- 快速通道：免逐條對照，確認「失敗測試→修正→綠燈」證據鏈；diff 超過 3 檔時判斷是否應升級通道。
- 輕量通道：逐條對照僅限本次增修的標準與其 edge cases。

**完成條件**：驗收清單全勾且已合併。

## 退回與熔斷

退回 Phase 2 的次數（P3→P2、P4→P2、P5→P2 合計）與 Phase 5→4 的打回次數，記在 PR 描述的「退回與打回」欄；開 PR 前發生的退回，開 PR 時補記。

- 同一 feature **累計退回 Phase 2 達 2 次** → 熔斷：停止流程，**強制拆分**——已想清楚的部分保留成獨立 spec 續行，只把糾纏不清的部分回 Phase 1 重新定義（有真人：拆分決策拉高為專案級討論；無真人：換 fresh agent 從問題定義主導拆分）。
- **Phase 5→4 打回達 2 次** → 第三次實作前強制先做根因分析並寫入 PR。

## 工具邊界（reference）

| 動作 | 用 | 附註 |
|---|---|---|
| 目標確認（分流前） | `steelman-requirement` 對提案 | bug 回報免；「需求背後的目標」帶入 spec 動機 |
| spec / plan / tasks | `/speckit-*` | 其他 plugin 的規劃類 skill（例如 Superpowers 的 `brainstorming`、`writing-plans`）不在本流程使用 |
| 規劃完成檢查、開 PR 前自我檢查 | `/speckit-analyze`、`/speckit-converge` | 不取代 Phase 5 的獨立驗收 |
| 錘鍊需求 | `grill-me` 對 spec 文件 | 產出只寫回 spec |
| 實作 | 主 agent 派 fresh-context subagent，依 Phase 4 | 其他 plugin 的執行類 skill（例如 Superpowers 的 `subagent-driven-development`）不在本流程使用 |
| 除錯 | 失敗測試＋根因修正；原因不清楚時用 `diagnosing-bugs` | 先重現、找到根因才修 |
| 品質門檻 | `.specify/memory/constitution.md` | speckit 各階段自動引用。本流程以原則名稱引用它，不寫條號；條號以專案自己的文件為準，名稱不同時以內容對應 |

## 失效模式（reference）

| 症狀 | 對策 |
|---|---|
| 把解決方案當目標，做完才發現不是 PM 要的 | 提案一律先過目標確認（`steelman-requirement`）；「bug」但 spec 未描述該期望行為 → 視為提案照跑 |
| AI 做出來的不是想要的 | 檢查 grill 執行者是否獨立於 spec 作者（非作者主持／fresh-context agent） |
| AI 回報完成但驗收失敗 | Phase 3 起驗收標準就要對應測試任務（`/speckit-analyze` 檢查）；開 PR 前跑 `/speckit-converge` |
| 流程被私下繞過，或分流越放越寬 | 沒有定期檢視 | 只能在 PR 驗收時發現；反覆出現就提 PR 修訂本流程或 constitution |
| spec 和程式碼對不上 | 執行「改需求必退 Phase 2」 |
| 同一 feature 反覆退回繞圈 | 觸發熔斷：強制拆分，糾纏部分回 Phase 1 重新定義 |
| 測試存在但形同虛設 | 以斷言為單位抽查「改壞行為、斷言須轉紅」；適用豁免就走豁免程序，不寫假測試 |
| 成員產出品質落差 | PR review 檢查 spec 的 grill 檢核與 commit 歷史 |

## 完整版文件

各階段負責人與代位規則、Phase 0 專案初始化、熔斷細節見 [full-workflow.md](full-workflow.md)。
