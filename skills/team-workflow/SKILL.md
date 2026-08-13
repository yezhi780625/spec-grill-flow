---
name: team-workflow
description: 團隊 AI 交辦任務流程（Spec-Kit + grill + Superpowers）。開始任何 feature、bug fix 或程式碼變更前先用本流程分流；撰寫或修改 spec、規劃任務、開 PR、驗收、合併後 retro 時也使用。
---

團隊交辦 AI 任務的流程。核心原則：**spec 文件是唯一真相源**——所有需求結論最終寫回 `specs/` 下的 spec 文件，未寫回的結論視為不存在。

```
分流 ─┬─ 完整通道：定義 → 錘鍊 → 規劃 → 實作 → 驗收 → Retro
      └─ 快速通道：────────────────→ 實作 → 驗收 → Retro
```

## 分流（每個任務的第一步）

任務**同時滿足**以下三條件走快速通道，否則走完整通道：

1. 不新增、不修改任何 spec 中的驗收標準或行為描述（bug fix 使程式碼回到 spec 描述的行為、重構、依賴升級、typo 都算符合）
2. 預估變更範圍在 3 個檔案以內
3. 不引入新依賴、不新增抽象層

**快速通道**＝直接進 Phase 4（TDD 照常強制：先寫重現問題的失敗測試）→ Phase 5 驗收（免 spec 逐條對照，reviewer 確認測試證明修正即可）→ Phase 6 retro。

**升級規則**：快速通道進行中一旦發現需要動 spec 的行為——立即停手，轉入完整通道從 Phase 1 或 Phase 2 開始。判斷猶豫時，一律走完整通道。

## Phase 1：定義

用 `/speckit-specify` 把原始需求產成 spec 草稿，commit 進 feature branch。

**完成條件**：spec 草稿已 commit。此時 spec 未經檢驗，直接進 Phase 3 是流程違規。

## Phase 2：錘鍊（grill）

用 `grill-me` 對著 Phase 1 的 spec 文件逼問，每一輪修正**直接改 spec 並 commit**——git log 就是訪談紀錄。追問至少涵蓋：

- 每條驗收標準怎麼量測？由誰／什麼工具判定通過？
- Edge cases 與失敗路徑列了嗎？
- 需求之間有無矛盾？
- 「不做什麼」明確了嗎？

grill 的產出只寫回 spec 本身（不產 ADR、glossary 或其他文件）。

**完成條件**：spec 頂部的 grill 檢核清單四項全勾：

```markdown
## Grill 檢核
- [ ] 所有驗收標準可量測
- [ ] Edge cases 與失敗路徑已列出
- [ ] 無互相矛盾的需求
- [ ] 「不做什麼」已明確
```

## Phase 3：規劃

`/speckit-plan` 產技術計畫 → `/speckit-tasks` 拆任務，一併 commit。過程中發現 spec 有洞 → 退回 Phase 2 補 grill 後再回來。

**完成條件**：每條驗收標準在 tasks 中有對應的測試任務。

## Phase 4：實作

依 tasks 逐項實作，走 Superpowers 紀律：

- TDD 強制：測試先寫、先失敗，才准實作。
- 除錯：先找根因，才准修。
- 實作中發現需求問題 → 停手，退回 Phase 2 改 spec、過 grill，再繼續。需求變更一律經過 spec——spec 與程式碼不同步是本流程最大的失效模式。

**完成條件**：所有 tasks 完成且測試綠燈。

## Phase 5：驗收

開 PR，reviewer（非實作者本人）對照 spec 逐條驗收：

```markdown
## Spec 驗收（對照 specs/xxx/spec.md）
- [ ] 每條驗收標準有證據（自動測試連結，或人工驗證步驟與結果）
- [ ] 所有 edge cases 有對應測試
- [ ] 未實作 spec 範圍外的功能（scope creep 檢查）
```

任一條不過 → 退回 Phase 4；根因在 spec → 退回 Phase 2。全過 → 合併。

快速通道的驗收：免 spec 逐條對照，reviewer 確認「失敗測試→修正→綠燈」的證據鏈即可。

**完成條件**：驗收清單全勾且已合併。

## Phase 6：Retro（合併後，2 分鐘內完成）

合併後立即在 `specs/retro-log.md` 回填一行（表格欄位：日期、feature、通道、打回次數、症狀分類、一句話備註）。症狀分類使用下方失效模式表的既有分類；出現新型症狀時，在失效模式表新增一列並在備註標注「新增」。

- 快速通道的任務也要回填——它的打回數據是「分流條件是否太寬」的偵測器。
- 零打回一次過的任務同樣回填，這是流程健康度的分母。

**完成條件**：retro-log 已回填。此行是 constitution 季度檢視與「是否引入 ECC reviewer」決策的數據來源。

## 工具邊界（reference）

| 動作 | 用 | 附註 |
|---|---|---|
| spec / plan / tasks | `/speckit-*` | Superpowers 的 /brainstorm、/write-plan 不在本流程使用 |
| 錘鍊需求 | `grill-me` 對 spec 文件 | 產出只寫回 spec |
| 實作、TDD、除錯 | Superpowers | — |
| 品質門檻 | `.specify/memory/constitution.md` | speckit 各階段自動引用 |

## 失效模式（reference）

| 症狀 | 對策 |
|---|---|
| 做出來的不是想要的 | Phase 2 grill 深度不足，檢查檢核清單是否流於形式 |
| 回報完成但驗收失敗 | Phase 3 起驗收標準就要對應測試任務 |
| spec 和程式碼對不上 | 執行「改需求必退 Phase 2」 |
| 成員產出品質落差 | PR review 檢查 spec 的 grill 檢核與 commit 歷史 |
| 快速通道打回率升高 | 分流條件太寬，收緊條件或調降檔案數上限 |
| 測試都過但仍有安全／架構問題 | 缺獨立驗收——引入 ECC reviewer，條件見 [full-workflow.md](full-workflow.md) 附錄 B |

## 完整版文件

各階段負責人、Phase 0 專案初始化步驟、ECC reviewer 引入條件（附錄 B）見 [full-workflow.md](full-workflow.md)。
