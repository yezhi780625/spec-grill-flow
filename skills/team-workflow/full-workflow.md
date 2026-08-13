# AI 交辦任務 Workflow（團隊版）

**工具鏈**：Spec-Kit（文件骨架）＋ grill-me（需求錘鍊）＋ Superpowers（實作紀律）
**第二階段候補**：ECC reviewer（獨立驗收，見附錄 B）

---

## 總覽

```
需求 → 分流 ─┬─ 完整通道：[P1 定義] → [P2 錘鍊] → [P3 規劃] → [P4 實作] → [P5 驗收] → [P6 Retro]
             │             speckit     grill-me    speckit     superpowers  spec 對照
             └─ 快速通道：──────────────────────────────────→ [P4 實作] → [P5 驗收] → [P6 Retro]
```

核心原則：**Spec-Kit 是唯一文件真相源**。所有需求、計畫、任務都活在 repo 的 spec 文件裡；grill 和實作過程的所有結論，最終都必須寫回 spec，否則視為不存在。

---

## 分流（每個任務的第一步）

任務**同時滿足**以下三條件走快速通道，否則走完整通道：

1. 不新增、不修改任何 spec 中的驗收標準或行為描述（bug fix 使程式碼回到 spec 描述的行為、重構、依賴升級、typo 都算符合）
2. 預估變更範圍在 3 個檔案以內
3. 不引入新依賴、不新增抽象層

**快速通道**＝直接進 Phase 4（TDD 照常強制）→ Phase 5 驗收（免 spec 逐條對照，reviewer 確認「失敗測試→修正→綠燈」證據鏈）→ Phase 6 retro。

**升級規則**：快速通道進行中一旦發現需要動 spec 的行為——立即停手，轉入完整通道。判斷猶豫時，一律走完整通道。此規則單向：只能快速升級為完整，不得反向降級。

---

## Phase 0：專案初始化（一次性）

1. `specify init` 建立專案骨架。
2. `/speckit-constitution` 定義專案原則，至少包含：
   - 核心函式必須附單元測試（測試先於實作）
   - 每條驗收標準必須可量測（可自動驗證優先）
   - Spec 未通過 grill 檢核不得進入 plan 階段
3. 安裝 Superpowers plugin；在 CLAUDE.md 明確寫入：
   - 「Spec 與 plan 一律使用 /speckit-* 指令，**禁用** /brainstorm 與 /write-plan」
   - 「Superpowers 僅負責實作階段（TDD、除錯、執行）」
4. 從 mattpocock-skills 只抽 `grill-me` 一個 skill 放入團隊的 `.claude/skills/`，**不安裝整包**（to-spec、tdd、code-review 等與現有工具職責重疊）。
5. 在 PR template 加入 spec 驗收清單區塊（見 Phase 5）。

---

## Phase 1：需求定義（Spec-Kit）

**負責人**：需求發起者（工程師或 PM）

1. 執行 `/speckit-specify`，把原始需求丟進去，產出 spec 草稿。
2. 草稿 commit 進 feature branch——**先有文件，再有討論**。

> 此時的 spec 只是「格式化的初稿」，品質未經檢驗，不得直接進入 Phase 3。

---

## Phase 2：需求錘鍊（grill-me 對著 spec 打）

**負責人**：需求發起者，對象是 Phase 1 的 spec 文件

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

1. `/speckit-plan` 產出技術計畫（架構選擇、依賴、資料流）。
2. `/speckit-tasks` 拆解為可交辦的任務清單。
3. Plan 與 tasks 一併 commit。若 plan 過程發現 spec 有洞 → **退回 Phase 2 補 grill**，改完 spec 再回來。

---

## Phase 4：實作（Superpowers 紀律）

**負責人**：實作者＋AI agent

1. 依 tasks 逐項交辦給 agent 實作。
2. 強制 TDD 循環（red-green-refactor）：測試必須先寫、先失敗，才准實作。這是 constitution 條款，agent 跳過即打回。
3. 遇到 bug 走 Superpowers 的除錯法：先找根因，才准修。
4. **實作中發現需求問題 → 停手，退回 Phase 2**，改 spec、過 grill，再繼續。禁止「順手改一下需求」——spec 和實作不同步是整個流程最大的失效模式。

---

## Phase 5：驗收（spec 對照）

**負責人**：PR reviewer（不得是實作者本人）

1. 開 PR，template 自動帶入該 spec 的驗收清單，reviewer 逐條打勾：

```markdown
## Spec 驗收（對照 specs/xxx/spec.md）
- [ ] 驗收標準 1：<自動測試連結或人工驗證方式>
- [ ] 驗收標準 2：...
- [ ] 所有 edge cases 有對應測試
- [ ] 未實作 spec 範圍外的功能（scope creep 檢查）
```

2. 每條驗收標準優先附上自動測試作為證據；無法自動化的，寫明人工驗證步驟與結果。
3. 任一條打不了勾 → 打回 Phase 4；若原因是 spec 本身有問題 → 退回 Phase 2。
4. 全數通過 → 合併。Spec 文件與程式碼同 repo 留存，作為未來變更的基準。

---

## Phase 6：Retro（合併後，2 分鐘內完成）

**負責人**：實作者

合併後立即在 `specs/retro-log.md` 回填一行，欄位：日期、feature、通道（完整／快速）、打回次數、症狀分類、一句話備註。症狀分類使用失效模式表的既有分類；新型症狀則在失效模式表新增一列並標注。

快速通道與零打回的任務同樣回填——前者偵測分流條件是否太寬，後者提供打回率的分母。retro-log 是 constitution 季度檢視與附錄 B「是否引入 ECC reviewer」門檻（>15%）的數據來源。

---

## 失效模式與對策

| 症狀 | 根因 | 對策 |
|---|---|---|
| AI 做出來的不是想要的 | Phase 2 grill 不夠深 | 檢查 grill 檢核清單是否流於形式；加追問維度 |
| AI 回報完成但驗收失敗 | Phase 4 測試沒覆蓋驗收標準 | 驗收標準在 Phase 3 就要對應到測試任務 |
| Spec 和程式碼對不上 | 實作中偷改需求 | 嚴格執行「改需求必退 Phase 2」 |
| 不同成員產出品質落差大 | 有人跳過 grill | PR review 檢查 spec 的 grill 檢核與 commit 歷史 |
| 快速通道打回率升高 | 分流條件太寬 | 收緊三條件或調降檔案數上限 |
| 測試都過但仍有安全／架構問題 | 缺獨立驗收 | 觸發附錄 B，引入 ECC reviewer |

---

## 附錄 A：工具邊界速查

| 動作 | 用什麼 | 禁用什麼 |
|---|---|---|
| 產 spec / plan / tasks | `/speckit-*` | Superpowers 的 /brainstorm、/write-plan |
| 錘鍊需求 | grill-me（對 spec 文件） | grill-with-docs（會另產文件） |
| 實作、TDD、除錯 | Superpowers | mattpocock 的 tdd／diagnosing-bugs |
| 驗收 | PR spec 對照清單 | — |

## 附錄 B：ECC reviewer 引入條件（第二階段）

跑滿一至兩個月後，若同時滿足：

1. 「回報完成但驗收失敗」比率仍高（建議門檻：>15% 的 PR 被打回），且
2. 失敗類型是**測試抓不到的**：安全漏洞、架構偏離、需求理解錯誤

則從 ECC 抽取對應的 reviewer（security-reviewer 或 language reviewer）作為 Phase 5 的前置自動關卡。屆時需明確：ECC reviewer 是「交付關卡」，Superpowers 內建 review 降級為「實作中自我檢查」，避免雙重審查互相矛盾。抽出的 agent/skill 指定一名 owner 負責定期同步上游更新。

若測試紀律已把問題壓下去——不要加。少一個來源永遠比多一層驗證好維運。
