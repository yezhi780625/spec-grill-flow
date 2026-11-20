---
description: 一次性初始化：在當前專案部署 spec-grill-flow 工作流（spec-kit、constitution、人力宣告與核准人、settings 分層 gitignore、CLAUDE.md、PR template）。
disable-model-invocation: true
---

在當前專案執行 spec-grill-flow 的 Phase 0 初始化。逐步執行，每步先偵測現況、冪等處理（已存在則跳過或詢問），全部完成後輸出總結表。

1. **前置檢查**：
   - 確認 `specify` CLI 可用且版本 ≥1.0（`specify version`）。不可用 → 給使用者安裝指令 `uv tool install specify-cli`（需 Python 3.11+，含 uv 本身的安裝方式）；版本低於 1.0 → 請使用者用原本的安裝工具升級：`uv tool install --force specify-cli` 或 `pipx install --force specify-cli`。可先跳過此步完成其餘步驟，最後在總結表標注待補。
2. **spec-kit 初始化**：
   - **repo 中不存在 `.specify/`** → 步驟 1 的 CLI 檢查是「待補」時，這一項也列「待補」，等 CLI 裝好後重跑 setup。CLI 可用 → 先請使用者 commit 或 stash 現有變更（初始化產生的檔案才看得出 diff），再執行 `specify init --here --force --integration claude`。兩個旗標都要帶：非互動執行時不帶 `--integration` 會預設安裝 GitHub Copilot 的檔案；非空目錄不帶 `--force` 會直接中止。專案有會改寫 Markdown 的 formatter（prettier 等，多半由 lint-staged 在 commit 時執行）時，徵詢後把 `.specify/` 與 `.claude/skills/speckit-*/` 加進它的忽略清單（prettier 是 `.prettierignore`）再 commit：spec-kit 以檔案雜湊判斷受管檔案有沒有被改過，被重排之後 `specify integration status` 會停在 WARNING；之後升級時，`.claude/skills/speckit-*` 的檔案被改過會擋下升級，`.specify/` 下的共用檔則被跳過、留著舊內容。
   - **已存在 `.specify/`** → 讀 `.specify/integration.json` 的 `version`。已是 1.x → 跳過。低於 1.0（或沒有這個檔）→ 這是既有專案的遷移，不在 setup 處理：**停止 setup**，請使用者執行 `/spec-grill-flow:migrate`，它會升級 spec-kit 並同步本 command 產出的檔案。
3. **部署 constitution**：`.specify/memory/constitution.md` 若為預設模板或不存在 → 以本 plugin `templates/constitution.md` 為底，詢問使用者填入 `<角括號>` 參數後寫入；已有客製化內容 → 不覆蓋，照下方「既有 constitution 的對照」處理。
4. **宣告人力與核准人**：詢問專案有無真人協作者（用不用人由分流決定——完整通道真人 grill、其餘 AI 代位，見 team-workflow），寫入 constitution 的 Governance 段，並填上 constitution 修訂核准人。無真人時為使用者本人。填實際的姓名，不留 placeholder。
5. **舊版產物**：下列任一項存在 → 這是 0.7.0 之前的 setup 留下的。總結表列為「待補」，請使用者執行 `/spec-grill-flow:migrate` 處理；setup 不動它們。都沒有 → 跳過。
   - `.claude/settings.json` 有 `"model": "opus"` 與 `"effortLevel": "high"`，或 `.claude/skills/speckit-plan/SKILL.md` 的 frontmatter 有 `model: fable`（0.5.0 之前寫入的模型設定）。
   - `specs/retro-log.md`（0.7.0 起流程不再使用）。
   - `.specify/extensions.yml` 的 `installed` 有 `git`（0.7.0 起分支由流程建立；migrate 會徵詢移除或保留）。
   - constitution 裡有 `早於或同於` 或 `檢視節奏雙保險`（0.7.0 改寫或移除的條文）。
   - `.specify/feature.json` 已被 git 追蹤（`git ls-files --error-unmatch .specify/feature.json`；spec-kit 1.x 把它當成每個人本機的狀態）。
6. **.gitignore 保護 settings 分層**：`.claude/settings.json` 是**追蹤中的團隊共用基準**（只放非敏感的共用鍵）；`.claude/settings.local.json` 是**個人覆寫**（模型偏好、權限允許規則），優先權更高，一律不進版控。此步**無條件執行**——權限允許規則與個人的模型偏好都會寫進 local 檔，任何專案都會長出它。判斷原則：偵測一律以 git 本身為準（`git check-ignore -v`），勿只做 `.gitignore` 字面比對——等效樣式（`.claude` 無斜線、`**/.claude/`、上層目錄的 .gitignore）字面比對抓不到，而已用負向樣式（`!.claude/settings.json`）修好的配置又會被誤判；也勿只看 `git status` 乾不乾淨——`-v` 會指出命中規則來自哪個檔案，若來自機器層全域 ignore（`core.excludesFile`、`~/.config/git/ignore`），那只保護該台機器，團隊其他成員沒有，repo 層仍須補上。**依序判斷**：
   - **衝突閘門**：`git check-ignore -v .claude/settings.json` 有輸出且命中規則在 repo 層 → **停下徵詢，且不得先 append**（團隊基準被忽略，補 local 那行只是無效噪音）：團隊共用的 settings.json 永遠進不了版控，建議改為只忽略 `settings.local.json`；使用者同意才調整，不擅自改既有規則。婉拒 → 總結表標注待補、跳過下一條的 append，**但最後一條（已追蹤檔案）檢查仍須執行**——它與 .gitignore 內容無關。
   - `git check-ignore .claude/settings.local.json` 無輸出，或僅命中機器層全域規則 → 在 repo 層 `.gitignore` append `.claude/settings.local.json`（repo 層已有等效規則則跳過，勿產生重複行）。
   - `.claude/settings.local.json` 已被 git 追蹤（`git ls-files --error-unmatch .claude/settings.local.json` 可查；`.gitignore` 對已追蹤檔案無效）→ 告知需執行 `git rm --cached .claude/settings.local.json`，由使用者自行執行，總結表標注待補。**此條無條件執行**，不受前兩條結果影響。
7. **CLAUDE.md**：段落以原則名稱引用 constitution。constitution 不是這次在步驟 3 以範本新建的 → 先確認「既有 constitution 的對照」已有結果（步驟 3 做過就沿用；由 migrate 執行本步驟時，照它的步驟 6 在這裡做）。接著將 `templates/claude-md-snippet.md` 的「交辦流程鐵律」段落 append 到專案 CLAUDE.md。已含該段落 → 與現行 template 比對內容：一致則跳過；不一致則徵詢是否更新為新版（更新時保留使用者在段落內自行加註的內容）——只看標題就跳過會讓存量專案永遠拿不到 template 的後續修訂。
8. **PR／MR template**：先判斷平台——專案有 `.gitlab-ci.yml` 或 `.gitlab/` 目錄，或 `git remote get-url origin` 的網址含 `gitlab` → GitLab，目標是 `.gitlab/merge_request_templates/Default.md`；有 `.github/` 目錄或網址含 `github` → GitHub，目標是 `.github/pull_request_template.md`；判斷不出來 → 詢問使用者。將 `templates/pr-template.md` 複製到目標位置。已存在 → 與 template 比對：內容相同則跳過，不同則詢問合併方式。GitLab 只在網頁上開 MR 時自動帶入範本，所以流程另外要求開 PR／MR 時以範本作為描述骨架。
9. **驗證與總結**：列出每一步的結果（新建／跳過／待補／備份後覆蓋；constitution 的對照另有「補上條文」「修訂條文」與「保留（使用者選擇）」），提示使用者 commit 這些變更。

規則：任何會覆蓋既有檔案的動作先備份並徵詢；與 template 比對時只看文字內容，不計空行、行尾空白與換行位置的差異（專案的 formatter 可能重排過）；本 command 可重複執行（例如本 plugin 升級後重跑，以同步 CLAUDE.md 段落與 PR template）。既有專案跨大版本升級走 `/spec-grill-flow:migrate`。

## 既有 constitution 的對照

本流程的 CLAUDE.md 段落與 skill 以原則名稱引用 constitution（例如 Test-First、根因除錯），不寫條號。專案已有自己的 constitution 時，這些引用要有對應的條文才有依據，所以寫 CLAUDE.md 段落之前先對照：

1. 逐條讀本 plugin `templates/constitution.md` 的 Core Principles I 到 VI，以**內容**判斷專案的 constitution 有沒有等效的規則。名稱或條號不同不算缺；只涵蓋一部分的記「部分」（例如有測試先行，沒有豁免程序）。
2. 把結果列成表給使用者看，每條一列：範本的原則、專案對應條文的條號與名稱、狀態（有／部分／沒有）。六條都是「有」、且沒有第 5 點的牴觸條文 → 記「跳過」，不改任何檔案。
3. 狀態是「部分」或「沒有」的，逐條徵詢是否補上。constitution 是專案自己的文件，使用者婉拒就原樣保留。
   - **同意** → 第一次修改前先確認有備份：constitution 已被 git 追蹤、且沒有未 commit 的修改（`git status --short .specify/memory/constitution.md` 無輸出）→ git 歷史就是備份，不另存檔；否則另存為 `constitution-backup-<YYYYMMDD-HHMMSS>.md`。舊版 setup 留下的 `constitution-backup.md` 是當時的內容，不能當成這次的備份，也不要覆蓋它。「沒有」的：把範本的該條附加在專案既有原則之後，用下一個可用的條號、保留範本的原則名稱；既有條文不重新編號。「部分」的：把缺的內容補進專案原本那一條，補上的句子先給使用者看過。結果記「補上條文」，更新 constitution 的修訂日期（文件有這個欄位時），並提醒修訂要經專案自己的核准程序。
   - **婉拒** → 記「保留（使用者選擇）」，重跑時會再問。婉拒的是 Test-First 時另外提醒：`/speckit-tasks` 只在 spec 或 constitution 要求時才產生測試任務（見 team-workflow 的 Phase 3），之後呼叫它要明確要求 TDD。
4. 不要為了對齊條號而改 CLAUDE.md 段落或 skill 的引用文字——它們寫的是原則名稱，由讀的人依內容對應到專案的條文。
5. **牴觸的條文**：對照時一併找出專案 constitution 裡禁止或限制本流程做法的條文。已知的例子是 commit（2026-11-20 一個實際專案的回報）：本流程以 commit 作為驗收證據（Phase 1 到 4 的 spec 草稿、grill 紀錄、測試先於實作、tasks 勾選），條文規定工具或流程不得自行 commit 時就會牴觸。找不到 → 記「無」。找到的逐條列給使用者：條文原句、牴觸的是流程的哪條規則。每一條由使用者決定，不自行修改：
   - **修訂條文** → 把條文的範圍改寫到兩者可以並存（例如：交辦給 agent 的任務，由 agent 在該 feature branch 內依流程 commit；工具的 hook 不自動 commit、不催促）。改寫前後的對照先給使用者確認才寫入，備份與修訂日期的做法同第 3 點。記「修訂條文」。
   - **保留** → 以 constitution 為準，流程在這一處讓步。記「保留（使用者選擇）」，並在總結寫明哪條流程規則在本專案不適用、Phase 5 因此少了哪一項證據。重跑時會再問。

**完成條件**：六條原則各有結果——有、補上條文、或保留（使用者選擇）；牴觸的條文各有結果——修訂條文、保留（使用者選擇），或記「無」。
