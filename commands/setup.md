---
description: 一次性初始化：在當前專案部署 spec-grill-flow 工作流（spec-kit、constitution、人力宣告與核准人、settings 分層 gitignore、CLAUDE.md、PR template）。
disable-model-invocation: true
---

在當前專案執行 spec-grill-flow 的 Phase 0 初始化。逐步執行，每步先偵測現況、冪等處理（已存在則跳過或詢問），全部完成後輸出總結表。

1. **前置檢查**：
   - 確認 `specify` CLI 可用且版本 ≥1.0（`specify version`）。不可用 → 給使用者安裝指令 `uv tool install specify-cli`（需 Python 3.11+，含 uv 本身的安裝方式）；版本低於 1.0 → 請使用者用原本的安裝工具升級：`uv tool install --force specify-cli` 或 `pipx install --force specify-cli`。可先跳過此步完成其餘步驟，最後在總結表標注待補。
2. **spec-kit 初始化**：
   - **repo 中不存在 `.specify/`** → 步驟 1 的 CLI 檢查是「待補」時，這一項也列「待補」，等 CLI 裝好後重跑 setup。CLI 可用 → 先請使用者 commit 或 stash 現有變更（初始化產生的檔案才看得出 diff），再執行 `specify init --here --force --integration claude`。兩個旗標都要帶：非互動執行時不帶 `--integration` 會預設安裝 GitHub Copilot 的檔案；非空目錄不帶 `--force` 會直接中止。
   - **已存在 `.specify/`** → 讀 `.specify/integration.json` 的 `version`。已是 1.x → 跳過。低於 1.0（或沒有這個檔）→ 這是既有專案的遷移，不在 setup 處理：**停止 setup**，請使用者執行 `/spec-grill-flow:migrate`，它會升級 spec-kit 並同步本 command 產出的檔案。
3. **部署 constitution**：`.specify/memory/constitution.md` 若為預設模板或不存在 → 以本 plugin `templates/constitution.md` 為底，詢問使用者填入 `<角括號>` 參數後寫入；若已有客製化內容 → 先備份為 `constitution-backup.md` 再詢問是否合併。
4. **宣告人力與核准人**：詢問專案有無真人協作者（用不用人由分流決定——完整通道真人 grill、其餘 AI 代位，見 team-workflow），寫入 constitution 的 Governance 段，並填上 constitution 修訂核准人。無真人時為使用者本人。填實際的姓名，不留 placeholder。
5. **舊版產物**：下列任一項存在 → 這是 0.7.0 之前的 setup 留下的。總結表列為「待補」，請使用者執行 `/spec-grill-flow:migrate` 處理；setup 不動它們。都沒有 → 跳過。
   - `.claude/settings.json` 有 `"model": "opus"` 與 `"effortLevel": "high"`，或 `.claude/skills/speckit-plan/SKILL.md` 的 frontmatter 有 `model: fable`（0.5.0 之前寫入的模型設定）。
   - `specs/retro-log.md`（0.7.0 起流程不再使用）。
   - constitution 裡有 `早於或同於` 或 `檢視節奏雙保險`（0.7.0 改寫或移除的條文）。
   - `.specify/feature.json` 已被 git 追蹤（`git ls-files --error-unmatch .specify/feature.json`；spec-kit 1.x 把它當成每個人本機的狀態）。
6. **.gitignore 保護 settings 分層**：`.claude/settings.json` 是**追蹤中的團隊共用基準**（只放非敏感的共用鍵）；`.claude/settings.local.json` 是**個人覆寫**（模型偏好、權限允許規則），優先權更高，一律不進版控。此步**無條件執行**——權限允許規則與個人的模型偏好都會寫進 local 檔，任何專案都會長出它。判斷原則：偵測一律以 git 本身為準（`git check-ignore -v`），勿只做 `.gitignore` 字面比對——等效樣式（`.claude` 無斜線、`**/.claude/`、上層目錄的 .gitignore）字面比對抓不到，而已用負向樣式（`!.claude/settings.json`）修好的配置又會被誤判；也勿只看 `git status` 乾不乾淨——`-v` 會指出命中規則來自哪個檔案，若來自機器層全域 ignore（`core.excludesFile`、`~/.config/git/ignore`），那只保護該台機器，團隊其他成員沒有，repo 層仍須補上。**依序判斷**：
   - **衝突閘門**：`git check-ignore -v .claude/settings.json` 有輸出且命中規則在 repo 層 → **停下徵詢，且不得先 append**（團隊基準被忽略，補 local 那行只是無效噪音）：團隊共用的 settings.json 永遠進不了版控，建議改為只忽略 `settings.local.json`；使用者同意才調整，不擅自改既有規則。婉拒 → 總結表標注待補、跳過下一條的 append，**但最後一條（已追蹤檔案）檢查仍須執行**——它與 .gitignore 內容無關。
   - `git check-ignore .claude/settings.local.json` 無輸出，或僅命中機器層全域規則 → 在 repo 層 `.gitignore` append `.claude/settings.local.json`（repo 層已有等效規則則跳過，勿產生重複行）。
   - `.claude/settings.local.json` 已被 git 追蹤（`git ls-files --error-unmatch .claude/settings.local.json` 可查；`.gitignore` 對已追蹤檔案無效）→ 告知需執行 `git rm --cached .claude/settings.local.json`，由使用者自行執行，總結表標注待補。**此條無條件執行**，不受前兩條結果影響。
7. **CLAUDE.md**：將 `templates/claude-md-snippet.md` 的「交辦流程鐵律」段落 append 到專案 CLAUDE.md。已含該段落 → 與現行 template 比對內容：一致則跳過；不一致則徵詢是否更新為新版（更新時保留使用者在段落內自行加註的內容）——只看標題就跳過會讓存量專案永遠拿不到 template 的後續修訂。
8. **PR template**：將 `templates/pr-template.md` 複製到 `.github/pull_request_template.md`（已存在則詢問合併方式）。
9. **驗證與總結**：列出每一步的結果（新建／跳過／待補／備份後覆蓋），提示使用者 commit 這些變更。

規則：任何會覆蓋既有檔案的動作先備份並徵詢；本 command 可重複執行（例如本 plugin 升級後重跑，以同步 CLAUDE.md 段落與 PR template）。既有專案跨大版本升級走 `/spec-grill-flow:migrate`。
