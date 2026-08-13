# AI 交辦任務 Workflow（團隊版）

**工具鏈**：Spec-Kit（文件骨架）＋ grill-me（需求錘鍊）＋ Superpowers（實作紀律）
**第二階段候補**：ECC reviewer（獨立驗收，見附錄 B）

> 本文件是流程的唯一長版真相源；[SKILL.md](SKILL.md) 是 agent 讀的濃縮版，兩者不一致時以本文件為準。

---

## 總覽

```
需求 → 分流 ─┬─ 完整通道：[P1 定義] → [P2 錘鍊] → [P3 規劃] → [P4 實作] → [P5 驗收] → [P6 Retro]
             │             speckit     grill-me    speckit     superpowers  spec 對照
             ├─ 輕量通道：[P1+P2 寫入即錘鍊] → [P3 簡化] → [P4 實作] → [P5 驗收] → [P6 Retro]
             └─ 快速通道：──────────────────────────────→ [P4 實作] → [P5 驗收] → [P6 Retro]
```

核心原則：**Spec-Kit 是唯一文件真相源**。所有需求、計畫、任務都活在 repo 的 spec 文件裡；grill 和實作過程的所有結論，最終都必須寫回 spec，否則視為不存在。

---

## 運作模式（Phase 0 宣告一次）

流程支援兩種模式，差別在「非本人」角色由誰擔任：

- **Team 模式**：「非作者」「非實作者」角色由真人成員擔任。
- **Solo 模式**（個人＋AI）：這些角色由 **fresh-context 的獨立 AI agent** 代位。關鍵不是換個名字，而是換個脈絡——代位 agent 只能讀文件產物（spec、diff、PR），**不得共享產生該產物的對話脈絡**，且 prompt 明確指定挑錯立場。參與過推理的 agent 會順著自己的假設附和，等於沒審。

各 Phase 的代位規則寫在該 Phase 內。

---

## 分流（每個任務的第一步）

依**任務性質**選通道——不需要先精讀 spec 預測差異。判斷猶豫時，一律往下選更完整的通道。

### 快速通道

任務屬於以下類型之一，且不引入新依賴、不新增抽象層：

- typo／文案修正
- 依賴升級
- bug fix（使程式碼回到 spec 已描述的行為）
- 不改外部行為的重構

「變更範圍約 3 個檔案」是**絆線而非資格門檻**：事前不強制精算；PR diff 實際超過 3 檔時，由 Phase 5 reviewer 判斷是否應升級通道，而非讓任務自動作廢。

流程：直接進 Phase 4（TDD 照常強制：先寫重現問題的失敗測試）→ Phase 5（免 spec 逐條對照，確認「失敗測試→修正→綠燈」證據鏈）→ Phase 6 retro。

### 輕量通道

需要動 spec，但只**新增或修改 1 條驗收標準**，且不引入新依賴、不新增抽象層。

流程：

1. **P1＋P2 合併（寫入即錘鍊）**：直接在 spec 增修該條標準並 commit；立刻只針對這條標準跑 grill 四維度各至少一問（單條標準數分鐘可完），修正再 commit。grill 執行者規則同 Phase 2（非作者／fresh agent）。
2. **P3 簡化**：在 tasks 補一條任務描述（含對應測試任務），免完整 plan。
3. **P4–P6 照常**。Phase 5 的逐條對照僅限本次增修的標準與其 edge cases。

### 完整通道

其餘一切：新 feature、牽動多條驗收標準、引入新依賴或抽象層、或任何拿不準的情況。走完整 Phase 1–6。

### 升級規則（單向）

任何通道進行中，一旦發現實際範圍超出該通道資格——立即停手，升級到對應通道（快速→輕量或完整；輕量→完整）。

**已寫的測試與程式碼不作廢**：全部帶入升級後的 Phase 4 繼續使用，只回頭補做 Phase 1–3 的 spec 與規劃工作。升級的成本是補文件，不是重來——低估通道等級不會讓你損失實作進度，所以不必為了保住進度而硬撐在原通道。

判斷猶豫時，一律升級。此規則單向：不得反向降級。

---

## Phase 0：專案初始化（一次性）

1. `specify init` 建立專案骨架。
2. `/speckit.constitution` 定義專案原則，至少包含：
   - 核心函式必須附單元測試（測試先於實作，豁免條款見 constitution I）
   - 每條驗收標準必須可量測（可自動驗證優先）
   - Spec 未通過 grill 檢核不得進入 plan 階段
3. 安裝 Superpowers plugin；在 CLAUDE.md 明確寫入：
   - 「Spec 與 plan 一律使用 /speckit.* 指令，**禁用** /brainstorm 與 /write-plan」
   - 「Superpowers 僅負責實作階段（TDD、除錯、執行）」
4. 從 mattpocock-skills 只抽 `grill-me` 一個 skill 放入團隊的 `.claude/skills/`，**不安裝整包**（to-spec、tdd、code-review 等與現有工具職責重疊）。
5. 在 PR template 加入 spec 驗收清單區塊（見 Phase 5）。
6. **宣告運作模式與 owner**：在 constitution 的 Governance 段寫明 Solo 或 Team 模式，並填齊三個 owner——constitution 修訂核准人、grill-me 上游同步 owner、（若日後引入）ECC agent owner。Solo 模式三者皆為本人；owner 的職責統一由 retro 檢視節奏觸發（見 Phase 6），不靠自覺。

---

## Phase 1：需求定義（Spec-Kit）

**負責人**：需求發起者（工程師或 PM）

1. 執行 `/speckit.specify`，把原始需求丟進去，產出 spec 草稿。
2. 草稿 commit 進 feature branch——**先有文件，再有討論**。

> 此時的 spec 只是「格式化的初稿」，品質未經檢驗，不得直接進入 Phase 3。

---

## Phase 2：需求錘鍊（grill-me 對著 spec 打）

**負責人**：**非 spec 作者**——與 Phase 5「非實作者 review」同構。需求誤解的成本高於實作瑕疵，自己 grill 自己寫的需求是流程最早、也最貴的盲點。

- **Team 模式**：由另一名成員主持 grill，spec 作者作答。
- **Solo 模式**：由 fresh-context 的獨立 subagent 執行 grill——只餵 spec 文件本身，不帶 Phase 1 的對話脈絡，prompt 明確指定 adversarial 立場；作者本人作答。

1. 開啟 `grill-me`，指定目標為該份 spec 文件，接受逼問。追問焦點至少涵蓋：
   - 每條驗收標準怎麼量測？由誰／什麼工具判定通過？
   - Edge cases 列了嗎？失敗路徑呢？
   - 需求之間有沒有互相矛盾？
   - 範圍邊界：明確「不做什麼」了嗎？
2. **每一輪修正直接改 spec 文件並 commit**——git log 就是訪談紀錄，reviewer 之後看 diff 就知道這份 spec 被錘過哪裡。
3. 完成後在 spec 文件頂部勾選 grill 檢核清單：

```markdown
## Grill 檢核
- [ ] 所有驗收標準可量測
- [ ] Edge cases 與失敗路徑已列出
- [ ] 無互相矛盾的需求
- [ ] 「不做什麼」已明確
```

**規則**：grill 的產出只能寫回 spec 本身，不另產 ADR／glossary／其他文件。

---

## Phase 3：規劃與拆解（Spec-Kit）

**負責人**：實作者

1. `/speckit.plan` 產出技術計畫（架構選擇、依賴、資料流）。
2. `/speckit.tasks` 拆解為可交辦的任務清單。
3. Plan 與 tasks 一併 commit。若 plan 過程發現 spec 有洞 → **退回 Phase 2 補 grill**，改完 spec 再回來（退回計數見「退回與熔斷」）。

---

## Phase 4：實作（Superpowers 紀律）

**負責人**：實作者＋AI agent

1. 依 tasks 逐項交辦給 agent 實作。
2. 強制 TDD 循環（red-green-refactor）：測試必須先寫、先失敗，才准實作。這是 constitution 條款，agent 跳過即打回。豁免類別（純 UI 樣式、一次性 migration、無邏輯膠水碼）走 constitution I 的豁免程序——附替代驗證方式，禁止空測試充數。
3. 遇到 bug 走 Superpowers 的除錯法：先找根因，才准修。
4. **實作中發現需求問題 → 停手，退回 Phase 2**，改 spec、過 grill，再繼續。禁止「順手改一下需求」——spec 和實作不同步是整個流程最大的失效模式。

---

## Phase 5：驗收（spec 對照）

**負責人**：PR reviewer——**不得是實作者本人**。Solo 模式：由 fresh-context 的獨立 agent 代位，只餵 PR diff 與 spec 文件，不帶實作過程的對話脈絡。

1. 開 PR，reviewer 將該 spec 的驗收標準貼入 PR 描述（或指示 agent 從 spec 生成），逐條打勾：

```markdown
## Spec 驗收（對照 specs/xxx/spec.md）
- [ ] 驗收標準 1：<自動測試連結或人工驗證方式>
- [ ] 驗收標準 2：...
- [ ] 所有 edge cases 有對應測試
- [ ] 未實作 spec 範圍外的功能（scope creep 檢查）
```

2. 每條驗收標準優先附上自動測試作為證據；無法自動化的，寫明人工驗證步驟與結果。抽查測試品質：revert 對應實作後測試必須轉紅（constitution I），空測試視同無測試。
3. 任一條打不了勾 → 打回 Phase 4；若原因是 spec 本身有問題 → 退回 Phase 2（計數見「退回與熔斷」）。
4. 快速通道 PR 加驗：實際 diff 超過 3 檔時，判斷是否應升級通道。
5. 全數通過 → 合併。Spec 文件與程式碼同 repo 留存，作為未來變更的基準。

---

## 退回與熔斷

退回路徑：P3→P2（plan 發現 spec 有洞）、P4→P2（實作發現需求問題）、P5→P4（驗收不過）、P5→P2（根因在 spec）。

退回不是免費的——反覆繞圈是「需求本身沒想清楚」的訊號，流程必須主動熔斷，而不是讓人在迴圈裡耗：

- **同一 feature 累計退回 Phase 2 達 2 次** → 熔斷：停止流程，回到 Phase 1 重寫或拆分該 feature。Team 模式：拉高為專案級討論；Solo 模式：換 fresh agent 從**問題定義**（而非現有 spec）重新開始。retro-log 症狀分類記「熔斷」。
- **Phase 5→4 打回達 2 次** → 第三次實作前，強制先做根因分析（constitution VI）並寫入 PR，才准再動手。

---

## Phase 6：Retro（合併後，2 分鐘內完成）

**負責人**：實作者

合併後立即在 `specs/retro-log.md` 回填一行，欄位：合併日、feature、通道（完整／輕量／快速）、開工日、退回次數（P3→P2＋P4→P2）、打回次數（P5→P4＋P5→P2）、症狀分類、一句話備註。症狀分類使用失效模式表的既有分類；新型症狀則在失效模式表新增一列並在備註標注「新增」。

- 快速與輕量通道的任務也要回填——它們的打回數據是「分流條件是否太寬」的偵測器。
- 零退回零打回的任務同樣回填，這是流程健康度的分母。
- 開工日＋合併日給出粗粒度 cycle time——退回／打回率下降但開工→合併天數持續上升，是「流程收太緊、交付變慢」的警訊，只看打回率看不到。

**檢視節奏（數據量驅動，取代日曆制）**：回填時數一下列數，每累積 **10 筆**觸發一次檢視，由回填該筆的人當週執行：

1. constitution 各原則是否攔過違規（連續兩次檢視未攔任何違規的原則，評估精煉或移除）
2. 分流條件是否太寬（快速／輕量通道打回率）或太緊（完整通道 cycle time）
3. grill-me 是否需對照上游同步（owner 見 Phase 0）
4. 是否達到附錄 B 的 ECC reviewer 引入條件

owner 不是頭銜，是「第 10n 筆觸發檢視」這個機制——低頻專案不會空轉，高頻專案不會拖到季末。

---

## 失效模式與對策

| 症狀 | 根因 | 對策 |
|---|---|---|
| AI 做出來的不是想要的 | Phase 2 grill 不夠深 | 檢查 grill 執行者是否獨立於 spec 作者（team：非作者主持；solo：fresh-context agent）——流於形式的檢核清單多半是自己審自己 |
| AI 回報完成但驗收失敗 | Phase 4 測試沒覆蓋驗收標準 | 驗收標準在 Phase 3 就要對應到測試任務 |
| Spec 和程式碼對不上 | 實作中偷改需求 | 嚴格執行「改需求必退 Phase 2」 |
| 同一 feature 反覆退回繞圈 | 需求根本沒想清楚 | 觸發熔斷：回 Phase 1 重寫或拆分（見「退回與熔斷」） |
| 測試存在但形同虛設 | TDD 被當表格應付、寫空測試交差 | Phase 5 抽查「revert 實作測試須轉紅」；適用豁免類別就走豁免程序，不寫假測試 |
| 不同成員產出品質落差大 | 有人跳過 grill | PR review 檢查 spec 的 grill 檢核與 commit 歷史 |
| 快速／輕量通道打回率升高 | 分流條件太寬 | 收緊任務類型白名單或輕量通道資格 |
| 打回率漂亮但交付變慢 | 流程收太緊 | 檢視 cycle time（開工日→合併日），放寬非關鍵關卡 |
| 測試都過但仍有安全／架構問題 | 缺獨立驗收 | 觸發附錄 B，引入 ECC reviewer |

---

## 附錄 A：工具邊界速查

| 動作 | 用什麼 | 禁用什麼 |
|---|---|---|
| 產 spec / plan / tasks | `/speckit.*` | Superpowers 的 /brainstorm、/write-plan |
| 錘鍊需求 | grill-me（對 spec 文件） | grill-with-docs（會另產文件） |
| 實作、TDD、除錯 | Superpowers | mattpocock 的 tdd／diagnosing-bugs |
| 驗收 | PR spec 對照清單 | — |

## 附錄 B：ECC reviewer 引入條件（第二階段）

retro-log 累積 **≥20 筆**後（與 Phase 6 檢視節奏同源），若同時滿足：

1. 「回報完成但驗收失敗」比率仍高（建議門檻：>15% 的 PR 被打回），且
2. 失敗類型是**測試抓不到的**：安全漏洞、架構偏離、需求理解錯誤

則從 ECC 抽取對應的 reviewer（security-reviewer 或 language reviewer）作為 Phase 5 的前置自動關卡。屆時需明確：ECC reviewer 是「交付關卡」，Superpowers 內建 review 降級為「實作中自我檢查」，避免雙重審查互相矛盾。抽出的 agent/skill 的 owner 於 Phase 0 已指定，負責在每次 retro 檢視時同步上游更新。

若測試紀律已把問題壓下去——不要加。少一個來源永遠比多一層驗證好維運。
