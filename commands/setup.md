---
description: 一次性初始化：在當前專案部署 spec-grill-flow 工作流（spec-kit、constitution、人力宣告與 owner、模型設定、settings 分層 gitignore、CLAUDE.md、PR template、retro-log）。
disable-model-invocation: true
---

在當前專案執行 spec-grill-flow 的 Phase 0 初始化。逐步執行，每步先偵測現況、冪等處理（已存在則跳過或詢問），全部完成後輸出總結表。

1. **前置檢查**：
   - 確認 `specify` CLI 可用（`uvx --from git+https://github.com/github/spec-kit.git specify check` 或已安裝的 specify）。不可用 → 給使用者安裝指令（含 uv 本身的安裝方式），可先跳過此步完成其餘步驟，最後在總結表標注待補。
   - 確認 Superpowers plugin 已載入（檢查 /brainstorm 等指令是否存在）。未載入 → 提示 plugin 依賴可能未解析，給手動安裝指令，同樣可先跳過。
2. **spec-kit 初始化**：repo 中不存在 `.specify/` → 引導執行 `specify init --here`；已存在則跳過。
3. **部署 constitution**：`.specify/memory/constitution.md` 若為預設模板或不存在 → 以本 plugin `templates/constitution.md` 為底，詢問使用者填入 `<角括號>` 參數後寫入；若已有客製化內容 → 先備份為 `constitution-backup.md` 再詢問是否合併。
4. **宣告人力與 owner**：詢問專案有無真人協作者（用不用人由分流決定——完整通道真人 grill、其餘 AI 代位，見 team-workflow），寫入 constitution 的 Governance 段，並填齊三個 owner（constitution 修訂核准人、grill-me 上游同步、ECC agent 若引入）。無真人時三者皆為使用者本人。**不得留 placeholder**——這是 retro 檢視機制的觸發依據。
5. **模型設定（建議配置，先徵詢再寫入）**：設計理由——spec 與規劃環節（speckit 三指令）用最強的 Fable（能力高於 Opus、單價約兩倍、速度較慢），因為需求想錯的成本高於實作瑕疵，且此環節 token 量小、以 `effort: medium` 控制花費；主對話量大，用 Opus。向使用者說明時勿描述為「快且便宜」。詢問使用者是否套用建議模型配置；同意才在 `.claude/settings.json` 寫入（合併，勿覆蓋既有鍵）`"model": "opus"`, `"effortLevel": "high"`，並對 spec-kit 產出的 `.claude/skills/speckit-specify/SKILL.md`、`speckit-plan/SKILL.md`、`speckit-tasks/SKILL.md` 的 frontmatter 加入四個鍵：`context: fork`、`background: false`、`model: fable`、`effort: medium`（已存在則跳過）。**必須有 `context: fork`**——實測 `model` 在 inline 執行時不會切換模型（與文件宣稱不符），只有 fork 進 subagent 時保證生效；`background: false` 讓主對話等待產出文件後再繼續（v2.1.218+ 起 fork 預設背景執行）。已知取捨：fork 後 skill 無法中途向使用者提問，但 speckit 三指令是單向產文件操作，影響有限。spec-kit ≥0.8.10 已將 custom commands 併入 skills；若專案是舊版 spec-kit 產出的 `.claude/commands/speckit*.md`，patch 該處（僅 `model`/`effort`）並建議升級。寫入前一併說明個人出口：`.claude/settings.json` 是**團隊共用基準**，個人要換模型在自己的 `.claude/settings.local.json` 蓋回去即可（優先權高於前者），與下一步的 ignore 成套；CI 不跑 Claude Code，不受影響。使用者婉拒或無對應模型存取權 → 跳過，流程照常可用。
6. **.gitignore 保護 settings 分層**：`.claude/settings.json` 是**追蹤中的團隊共用基準**（只放非敏感的共用鍵，如 model／effortLevel）；`.claude/settings.local.json` 是**個人覆寫**（模型偏好、權限允許規則），優先權更高，一律不進版控。此步**無條件執行**，不依附步驟 5 是否套用模型設定——權限允許規則與模型設定無關，任何專案都會長出 local 檔。判斷原則：偵測一律以 git 本身為準（`git check-ignore -v`），勿只做 `.gitignore` 字面比對——等效樣式（`.claude` 無斜線、`**/.claude/`、上層目錄的 .gitignore）字面比對抓不到，而已用負向樣式（`!.claude/settings.json`）修好的配置又會被誤判；也勿只看 `git status` 乾不乾淨——`-v` 會指出命中規則來自哪個檔案，若來自機器層全域 ignore（`core.excludesFile`、`~/.config/git/ignore`），那只保護該台機器，團隊其他成員沒有，repo 層仍須補上。**依序判斷**：
   - **衝突閘門**：`git check-ignore -v .claude/settings.json` 有輸出且命中規則在 repo 層 → **停下徵詢，且不得先 append**（團隊基準被忽略，補 local 那行只是無效噪音）：步驟 5 寫入的團隊基準永遠進不了版控，建議改為只忽略 `settings.local.json`；使用者同意才調整，不擅自改既有規則。婉拒 → 總結表標注待補、跳過下一條的 append，**但最後一條（已追蹤檔案）檢查仍須執行**——它與 .gitignore 內容無關。
   - `git check-ignore .claude/settings.local.json` 無輸出，或僅命中機器層全域規則 → 在 repo 層 `.gitignore` append `.claude/settings.local.json`（repo 層已有等效規則則跳過，勿產生重複行）。
   - `.claude/settings.local.json` 已被 git 追蹤（`git ls-files --error-unmatch .claude/settings.local.json` 可查；`.gitignore` 對已追蹤檔案無效）→ 告知需執行 `git rm --cached .claude/settings.local.json`，由使用者自行執行，總結表標注待補。**此條無條件執行**，不受前兩條結果影響。
7. **CLAUDE.md**：將 `templates/claude-md-snippet.md` 的「交辦流程鐵律」段落 append 到專案 CLAUDE.md。已含該段落 → 與現行 template 比對內容：一致則跳過；不一致則徵詢是否更新為新版（更新時保留使用者在段落內自行加註的內容）——只看標題就跳過會讓存量專案永遠拿不到 template 的後續修訂。
8. **PR template**：將 `templates/pr-template.md` 複製到 `.github/pull_request_template.md`（已存在則詢問合併方式）。
9. **Retro log**：建立 `specs/retro-log.md`（以 `templates/retro-log.md` 為底，已存在則跳過）。
10. **驗證與總結**：列出每一步的結果（新建／跳過／待補／備份後覆蓋），提示使用者 commit 這些變更；若套用了模型設定，建議團隊成員各自跑一次 `/model fable` 確認帳號有 Fable 存取權。

規則：任何會覆蓋既有檔案的動作先備份並徵詢；本 command 可重複執行（例如 spec-kit 升級後重跑以恢復 frontmatter patch）。
