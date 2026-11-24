# constitution 的舊條文

0.7.0 之前的 `templates/constitution.md` 有三處條文在 0.7.0 改寫或移除。constitution 是專案自己的文件，所以只處理與下表「舊條文」相同的句子（差別只在已填入的姓名也算相同）；改寫過的句子視為專案自己的決定，總結表列為「保留（團隊設定）」。

每一項給使用者看前後對照，同意才改；婉拒列為「保留（使用者選擇）」。

| 位置 | 舊條文（辨識用） | 現行寫法 | 為什麼改 |
|---|---|---|---|
| I. Test-First 的 Verification | `PR 中測試的 commit 早於或同於實作 commit` | `templates/constitution.md` 的 I. Verification 整句：新行為看 commit 順序，事後新增或修改的斷言看失敗證據，抽查以斷言為單位 | 測試與實作放在同一個 commit 時，事後無法驗證測試先行；整段 revert 的抽查擋不住永遠成立的斷言 |
| Governance 的 Owner 行 | 以 `**Owner**（Phase 0 填齊，不留空）：constitution 修訂核准人` 開頭，列出 grill-me 上游同步與 ECC agent 的 owner，結尾是 `職責由 retro 檢視節奏觸發。` | `**修訂核准人**（專案初始化時填上，不留空）：<原本填的核准人姓名>。無真人協作者時為本人。` | grill-me 的上游同步由 plugin repo 的 CI 處理；ECC 與 retro 已從流程移除 |
| Governance 的檢視節奏行 | 以 `檢視節奏雙保險：retro-log 每累積 10 筆` 開頭的整行 | 刪除這一行 | retro 已從流程移除 |

有任何一項修改時：

- 修訂程序照 `commands/setup.md` 的「既有 constitution 的對照」第 3 點「修訂照專案自己的 Governance 進行」：版號、修訂日期、Sync Impact Report 都依專案的 Governance。同一次執行裡已經做過對照的修改時，兩者併為同一次修訂，版號只升一次。
- 在總結裡提醒：修訂 constitution 要經核准人核准，遷移 PR 的 review 就是這個關卡。

**完成條件**：三項各有結果——完成（已更新）、保留（團隊設定）、保留（使用者選擇），或跳過（已是現行寫法；修訂核准人那一行寫「Phase 0 填上」的是 0.7.0 到 0.7.2 的版本，也算）。保留（使用者選擇）的項目重跑時會再問一次；要停止詢問，把那一句改成專案自己的寫法。
