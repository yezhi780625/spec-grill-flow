# 舊版模型設定的清理規則

0.5.0 之前的 setup 會把模型與 effort 寫進專案。本流程現在不指定這兩者，交給使用者選擇（理由見 team-workflow 的「模型與 effort」），所以遷移時清掉。

## 規則

- 逐項偵測。**值與下列舊版寫入值相同**才列出、說明影響並徵詢是否移除。值不同視為團隊自己的設定：保留，總結表列為「保留（團隊設定）」。
- 使用者選擇保留舊版值時照辦，總結表列為「保留（使用者選擇）」。重跑遷移時會再問一次；改成其他值就不再詢問。
- 備份：檔案已進版控且沒有未 commit 的修改，以 git 作為備份。其餘情況先在同目錄複製一份 `<原檔名>.pre-0.5.bak`，總結時提醒使用者確認後刪除、不要 commit。
- 保留下來的 speckit skill 鍵會讓 `specify integration upgrade` 把該檔視為本地修改而停下，處理方式見 SKILL.md 步驟 4。

## 偵測對象

- **`.claude/settings.json` 的 `"model"`、`"effortLevel"`**（舊版寫入 `"model": "opus"`、`"effortLevel": "high"`）：這是全隊共用的基準，會蓋掉每個人的模型選擇；頂層 `effortLevel` 還會套用到所有模型。建議移除，個人偏好改寫在自己的 `.claude/settings.local.json`。只動這兩個鍵，其他既有鍵勿覆蓋。
- **speckit 三指令 frontmatter 的 `model`、`effort`、`context: fork`、`background: false`**（四個鍵當成一組判斷：只有 `model`、`effort` 都是舊版值或不存在時，才提議移除這四個鍵；`model` 或 `effort` 是其他值時，四個鍵全部視為團隊設定保留——少了 `context: fork`，frontmatter 的 `model` 不會生效。檔案為 spec-kit 產出的 `.claude/skills/speckit-specify/SKILL.md`、`speckit-plan/SKILL.md`、`speckit-tasks/SKILL.md`；舊版一律寫入 `model: fable`、`effort: medium`、`context: fork`、`background: false`）：影響是這三個指令固定以 medium effort 的 Fable 執行，不跟隨使用者的選擇；依訂閱方案，Fable 可能在方案額度外另外計費；fork 後指令無法中途向使用者提問。只移除這四個鍵，frontmatter 其餘內容不動。
- **舊版 spec-kit 的 `.claude/commands/speckit*.md` 中的 `model`、`effort`**：同上處理，0.8.10 起 custom commands 已併入 skills，升級後這些檔案會被 skill 取代。
