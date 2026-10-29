---
description: 一次性初始化：在當前專案部署 spec-grill-flow 工作流（spec-kit、constitution、人力宣告與 owner、清理舊版模型設定、settings 分層 gitignore、CLAUDE.md、PR template、retro-log）。
disable-model-invocation: true
---

在當前專案執行 spec-grill-flow 的 Phase 0 初始化。逐步執行，每步先偵測現況、冪等處理（已存在則跳過或詢問），全部完成後輸出總結表。

1. **前置檢查**：
   - 確認 `specify` CLI 可用（`uvx --from git+https://github.com/github/spec-kit.git specify check` 或已安裝的 specify）。不可用 → 給使用者安裝指令（含 uv 本身的安裝方式），可先跳過此步完成其餘步驟，最後在總結表標注待補。
   - 確認 Superpowers plugin 已載入（檢查 /brainstorm 等指令是否存在）。未載入 → 提示 plugin 依賴可能未解析，給手動安裝指令，同樣可先跳過。
2. **spec-kit 初始化**：repo 中不存在 `.specify/` → 引導執行 `specify init --here`；已存在則跳過。
3. **部署 constitution**：`.specify/memory/constitution.md` 若為預設模板或不存在 → 以本 plugin `templates/constitution.md` 為底，詢問使用者填入 `<角括號>` 參數後寫入；若已有客製化內容 → 先備份為 `constitution-backup.md` 再詢問是否合併。
4. **宣告人力與 owner**：詢問專案有無真人協作者（用不用人由分流決定——完整通道真人 grill、其餘 AI 代位，見 team-workflow），寫入 constitution 的 Governance 段，並填齊三個 owner（constitution 修訂核准人、grill-me 上游同步、ECC agent 若引入）。無真人時三者皆為使用者本人。**不得留 placeholder**——這是 retro 檢視機制的觸發依據。
5. **清理舊版模型設定**：本流程不指定模型與 effort，交給使用者選擇（理由與規則見 team-workflow 的「模型與 effort」）。舊版 setup 曾寫入下列設定：逐項偵測，存在就列出並說明影響，徵詢後才移除，不得靜默刪除；都不存在就跳過。
   - **`.claude/settings.json` 的 `"model"`、`"effortLevel"`**（舊版寫入 `"model": "opus"`、`"effortLevel": "high"`）：這是全隊共用的基準，會蓋掉每個人的模型選擇；頂層 `effortLevel` 還會套用到所有模型。建議移除，個人偏好改寫在自己的 `.claude/settings.local.json`。使用者表示這是團隊刻意的選擇時，保留不動。只動這兩個鍵，其他既有鍵勿覆蓋。
   - **speckit 三指令 frontmatter 的 `model`、`effort`、`context: fork`、`background: false`**（檔案為 spec-kit 產出的 `.claude/skills/speckit-specify/SKILL.md`、`speckit-plan/SKILL.md`、`speckit-tasks/SKILL.md`；舊版一律寫入 `model: fable`、`effort: medium`、`context: fork`、`background: false`）：影響是這三個指令固定以 medium effort 的 Fable 執行，不跟隨使用者的選擇；Pro 方案的 Fable 另以 usage credits 計費；fork 後指令無法中途向使用者提問。只移除這四個鍵，frontmatter 其餘內容不動。
   - **舊版 spec-kit 的 `.claude/commands/speckit*.md` 中的 `model`、`effort`**：同上處理，並建議升級 spec-kit（≥0.8.10 已將 custom commands 併入 skills）。
6. **.gitignore 保護 settings 分層**：`.claude/settings.json` 是**追蹤中的團隊共用基準**（只放非敏感的共用鍵）；`.claude/settings.local.json` 是**個人覆寫**（模型偏好、權限允許規則），優先權更高，一律不進版控。此步**無條件執行**——權限允許規則與個人的模型偏好都會寫進 local 檔，任何專案都會長出它。判斷原則：偵測一律以 git 本身為準（`git check-ignore -v`），勿只做 `.gitignore` 字面比對——等效樣式（`.claude` 無斜線、`**/.claude/`、上層目錄的 .gitignore）字面比對抓不到，而已用負向樣式（`!.claude/settings.json`）修好的配置又會被誤判；也勿只看 `git status` 乾不乾淨——`-v` 會指出命中規則來自哪個檔案，若來自機器層全域 ignore（`core.excludesFile`、`~/.config/git/ignore`），那只保護該台機器，團隊其他成員沒有，repo 層仍須補上。**依序判斷**：
   - **衝突閘門**：`git check-ignore -v .claude/settings.json` 有輸出且命中規則在 repo 層 → **停下徵詢，且不得先 append**（團隊基準被忽略，補 local 那行只是無效噪音）：團隊共用的 settings.json 永遠進不了版控，建議改為只忽略 `settings.local.json`；使用者同意才調整，不擅自改既有規則。婉拒 → 總結表標注待補、跳過下一條的 append，**但最後一條（已追蹤檔案）檢查仍須執行**——它與 .gitignore 內容無關。
   - `git check-ignore .claude/settings.local.json` 無輸出，或僅命中機器層全域規則 → 在 repo 層 `.gitignore` append `.claude/settings.local.json`（repo 層已有等效規則則跳過，勿產生重複行）。
   - `.claude/settings.local.json` 已被 git 追蹤（`git ls-files --error-unmatch .claude/settings.local.json` 可查；`.gitignore` 對已追蹤檔案無效）→ 告知需執行 `git rm --cached .claude/settings.local.json`，由使用者自行執行，總結表標注待補。**此條無條件執行**，不受前兩條結果影響。
7. **CLAUDE.md**：將 `templates/claude-md-snippet.md` 的「交辦流程鐵律」段落 append 到專案 CLAUDE.md。已含該段落 → 與現行 template 比對內容：一致則跳過；不一致則徵詢是否更新為新版（更新時保留使用者在段落內自行加註的內容）——只看標題就跳過會讓存量專案永遠拿不到 template 的後續修訂。
8. **PR template**：將 `templates/pr-template.md` 複製到 `.github/pull_request_template.md`（已存在則詢問合併方式）。
9. **Retro log**：建立 `specs/retro-log.md`（以 `templates/retro-log.md` 為底，已存在則跳過）。
10. **驗證與總結**：列出每一步的結果（新建／跳過／待補／備份後覆蓋），提示使用者 commit 這些變更；若步驟 5 移除了舊版模型設定，提醒使用者告知團隊成員：speckit 三指令之後跟隨各自選擇的模型，個人偏好寫在 `.claude/settings.local.json`。

規則：任何會覆蓋既有檔案的動作先備份並徵詢；本 command 可重複執行（例如 spec-kit 升級後，或本 plugin 的 template 更新後重跑以同步）。
