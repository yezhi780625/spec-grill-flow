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
5. **模型設定（建議配置，先徵詢再寫入）**：設計理由——模型強度跟著「想錯的代價」配，不跟著 token 量配。spec、技術計畫與 grill 想錯的成本高於實作瑕疵，且 token 量小，用該方案內最強的模型與 effort high；tasks 是把已定案的 plan 拆成步驟，屬執行層，降一級；主對話（分流、實作）量大，用 Opus／Sonnet 的預設 effort（medium）。向使用者說明時勿把任何一檔描述為「快且便宜」。
   - **先問訂閱方案**，因為 Fable 的計費方式因方案而異（2026-09 的資訊，整理自第三方文章而非 Anthropic 官方頁面；套用前以官方定價頁與帳號用量頁面為準）：**Max** 的 Fable 含在週額度內，但最多佔一半，且消耗比其他模型快；**Pro** 的 Fable 從第一次請求就以 usage credits 另外計費（單價約為 Opus 的 2.5 倍），方案內含的最強模型是 Opus。**團隊裡只要有一位 Max 成員，共用基準就用 Max 配置**，Pro 成員以個人出口把 `fable` 改指向 Opus（見本步驟末），Max 成員因此不被拖到 Opus。全員都是 Pro 才套 Pro 配置。
   - **配置表**（使用者同意才寫入；婉拒 → 跳過，流程照常可用）：

     | 寫入位置 | Max | Pro |
     |---|---|---|
     | `.claude/settings.json`（主對話） | `"model": "opus"` | `"model": "sonnet"` |
     | `.claude/agents/spec-challenger.md`（上游代位 subagent） | `model: fable`, `effort: high` | `model: opus`, `effort: high` |
     | `speckit-specify`、`speckit-plan` 的 frontmatter | `model: fable`, `effort: high` | `model: opus`, `effort: high` |
     | `speckit-tasks` 的 frontmatter | `model: opus`, `effort: medium` | `model: sonnet`, `effort: medium` |

     Pro 使用者若明確表示願意為大功能的 plan 付 credits，可只把 `speckit-plan` 設為 `model: fable`，其餘維持 Pro 欄。
   - **`settings.json` 不寫 `effortLevel`**：頂層 `effortLevel` 會套用到所有模型，包括真人 grill 時用 `/model fable` 切過去的 Fable，會把它從預設的 high 壓低（v2.1.293 實測：`"effortLevel": "medium"` 時 Fable 的有效 effort 是 medium，拿掉後回到 high；Opus 5.5、Sonnet 5.5 的預設本來就是 medium）。已存在 `effortLevel`（例如舊版 setup 寫入的 `"high"`）→ 說明上述影響，建議移除，徵詢後才刪。
   - **`spec-challenger` 定義檔**：以本 plugin `templates/spec-challenger.md` 為底，把 `model` 換成配置表的值，寫到 `.claude/agents/spec-challenger.md`（進版控，全隊共用）。AI 代位 grill 與熔斷拆分的代位都以 `subagent_type: spec-challenger` 派出、**不另外指定 model**——派出時指定的 model 會蓋過定義檔。`effort: high` 必須寫在定義檔：沒寫時 subagent 的 effort 會受專案 `effortLevel` 影響（實測：專案有 `"effortLevel": "medium"` 且定義檔沒寫 effort → medium；定義檔寫 high → high）。
   - **寫入方式與比對規則**：`.claude/settings.json` 合併寫入，只動 `model`，其他既有鍵勿覆蓋。speckit frontmatter 寫在 spec-kit 產出的 `.claude/skills/speckit-specify/SKILL.md`、`speckit-plan/SKILL.md`、`speckit-tasks/SKILL.md`，除上表的 `model`、`effort` 外一律加上 `context: fork`、`background: false`。**所有寫入位置適用同一條比對規則**：鍵已存在且值與所選配置相同 → 跳過；值不同（例如舊版 setup 寫入的三指令一律 `model: fable`／`effort: medium`）→ 列出差異，徵詢後才更新，不得靜默保留也不得靜默覆蓋。`spec-challenger.md` 的內文與 template 不同時同樣列出差異、徵詢。唯一例外：選 Pro 配置、`speckit-specify` 已是 `model: opus`、`speckit-plan` 是 `model: fable` → 視為先前選定的單點升級，`speckit-plan` 的 `model` 算相符、不再詢問；`speckit-specify` 不是 `model: opus` 時，`speckit-plan` 的 `fable` 是舊版設定留下的，照「值不同」處理。
   - **必須有 `context: fork`**——實測 `model` 在 inline 執行時不會切換模型（與文件宣稱不符），只有 fork 進 subagent 時保證生效；`background: false` 讓主對話等待產出文件後再繼續（v2.1.218+ 起 fork 預設背景執行）。已知取捨：fork 後 skill 無法中途向使用者提問，但 speckit 三指令是單向產文件操作，影響有限。spec-kit ≥0.8.10 已將 custom commands 併入 skills；若專案是舊版 spec-kit 產出的 `.claude/commands/speckit*.md`，patch 該處（僅 `model`/`effort`）並建議升級。
   - **setup 寫不進檔案的環節**：真人主持的 grill 與 steelman 是 inline 互動，由使用者手動 `/model` 切換（agent 無法自己切）；代位 steelman 與 Phase 5 代位 reviewer 在派出時指定 `opus`。對照表見 team-workflow 的「模型配置」，寫入後向使用者指出這一段。
   - **個人出口（方案不一的團隊靠這個）**：共用檔寫的是團隊基準，個人差異不進版控。步驟 7 寫入專案 CLAUDE.md 的段落附有 Pro 成員的設定摘要，之後才加入、不會跑 setup 的成員也看得到。
     - **主對話模型**：寫在自己的 `.claude/settings.local.json`（優先權高於 `settings.json`，與下一步的 ignore 成套）。Pro 成員在 Max 基準的專案裡加 `"model": "sonnet"`。
     - **所有用到 `fable` 的環節**（`speckit-specify`／`speckit-plan` 的 fork、`spec-challenger` 代位、真人 grill 的 `/model fable`）：共用檔沒有個人層的覆寫，但 `fable` 別名解析成哪個模型可由環境變數 `ANTHROPIC_DEFAULT_FABLE_MODEL` 決定。Pro 成員設為 `claude-opus-5-5` 後，上述環節都改用 Opus，不觸發 credits 計費。設法：
       - **shell 設定檔**（`~/.zshrc`、`~/.bashrc`）加 `export ANTHROPIC_DEFAULT_FABLE_MODEL=claude-opus-5-5`：對自己在本機啟動的所有 Claude Code 生效。
       - **專案的 `.claude/settings.local.json`** 加 `"env": {"ANTHROPIC_DEFAULT_FABLE_MODEL": "claude-opus-5-5"}`：只影響本專案，且只在從終端機啟動時有效——由宿主管理 provider 的工作階段會忽略 settings 檔裡的這個變數（debug 紀錄會有 `Ignoring ANTHROPIC_DEFAULT_FABLE_MODEL from localSettings`）。
       - 兩者並存時，從終端機啟動以 `settings.local.json` 的值為準。所以設了全域 shell 變數、又想在某個 Pro 配置專案做 `speckit-plan` 的單點升級，就在該專案的 `settings.local.json` 把它設回 `claude-fable-5-1`；不這麼做，單點升級和 `/model fable` 都會被靜默改成 Opus。
       - **雲端工作階段兩種設法都到不了**：雲端的 clone 裡沒有 `settings.local.json`，也沒有本機的 shell 設定檔，但共用的 frontmatter 與 `spec-challenger.md` 照樣生效。可在雲端環境的環境變數設定加這個變數（未經實測）；用下方指令驗證通過前，Pro 成員在雲端不要跑 `/speckit-specify`、`/speckit-plan` 或代位 grill。
     - **兩個會讓改指向失效的寫法**：變數值要用完整模型 ID，寫別名 `opus` 時 fork 有效但 `/model fable` 會報 `unrecognized_model`；共用檔要寫別名 `fable`，寫成完整 ID（`claude-fable-5-1`）就不會被改指向。Opus 出新版時這個變數值要自己更新。
     - **驗證（不送出模型請求、不計費）**：在專案目錄、用平常啟動 Claude Code 的同一個 shell 跑 `claude -p "/cost" --model fable --output-format stream-json --verbose | grep -o '"model":"[^"]*"' | head -1`。輸出 `"model":"claude-opus-5-5"` 代表改指向生效；輸出 Fable 的模型 ID 代表沒生效，先不要跑用到 `fable` 的環節。原理：`/cost` 是本機指令，不呼叫模型，但輸出的 `init` 訊息會列出 `fable` 實際解析成的模型。**不要改用一般提示詞驗證**：`-p` 模式下 Fable 請求若會以 usage credits 計費，Claude Code 不會先詢問就直接計費。
     - 以上為 v2.1.286–2.1.293 實測：測試專案模擬 Max 與 Pro 兩種設定，重複執行 speckit 的 fork、`spec-challenger` 派出與驗證指令，Pro 設定下沒有任何一次用到 Fable。Max 的 Fable 用量達上限後的行為未經實測——屆時同樣可用上述環境變數暫時改指向 Opus，不必動共用檔。CI 不跑 Claude Code，不受影響。
6. **.gitignore 保護 settings 分層**：`.claude/settings.json` 是**追蹤中的團隊共用基準**（只放非敏感的共用鍵，如 model）；`.claude/settings.local.json` 是**個人覆寫**（模型偏好、權限允許規則），優先權更高，一律不進版控。此步**無條件執行**，不依附步驟 5 是否套用模型設定——權限允許規則與模型設定無關，任何專案都會長出 local 檔。判斷原則：偵測一律以 git 本身為準（`git check-ignore -v`），勿只做 `.gitignore` 字面比對——等效樣式（`.claude` 無斜線、`**/.claude/`、上層目錄的 .gitignore）字面比對抓不到，而已用負向樣式（`!.claude/settings.json`）修好的配置又會被誤判；也勿只看 `git status` 乾不乾淨——`-v` 會指出命中規則來自哪個檔案，若來自機器層全域 ignore（`core.excludesFile`、`~/.config/git/ignore`），那只保護該台機器，團隊其他成員沒有，repo 層仍須補上。**依序判斷**：
   - **衝突閘門**：`git check-ignore -v .claude/settings.json` 有輸出且命中規則在 repo 層 → **停下徵詢，且不得先 append**（團隊基準被忽略，補 local 那行只是無效噪音）：步驟 5 寫入的團隊基準永遠進不了版控，建議改為只忽略 `settings.local.json`；使用者同意才調整，不擅自改既有規則。婉拒 → 總結表標注待補、跳過下一條的 append，**但最後一條（已追蹤檔案）檢查仍須執行**——它與 .gitignore 內容無關。
   - `git check-ignore .claude/settings.local.json` 無輸出，或僅命中機器層全域規則 → 在 repo 層 `.gitignore` append `.claude/settings.local.json`（repo 層已有等效規則則跳過，勿產生重複行）。
   - `.claude/settings.local.json` 已被 git 追蹤（`git ls-files --error-unmatch .claude/settings.local.json` 可查；`.gitignore` 對已追蹤檔案無效）→ 告知需執行 `git rm --cached .claude/settings.local.json`，由使用者自行執行，總結表標注待補。**此條無條件執行**，不受前兩條結果影響。
7. **CLAUDE.md**：將 `templates/claude-md-snippet.md` 的「交辦流程鐵律」段落 append 到專案 CLAUDE.md。已含該段落 → 與現行 template 比對內容：一致則跳過；不一致則徵詢是否更新為新版（更新時保留使用者在段落內自行加註的內容）——只看標題就跳過會讓存量專案永遠拿不到 template 的後續修訂。
8. **PR template**：將 `templates/pr-template.md` 複製到 `.github/pull_request_template.md`（已存在則詢問合併方式）。
9. **Retro log**：建立 `specs/retro-log.md`（以 `templates/retro-log.md` 為底，已存在則跳過）。
10. **驗證與總結**：列出每一步的結果（新建／跳過／待補／備份後覆蓋），提示使用者 commit 這些變更；若套用了 Max 配置，提醒使用者告知團隊裡的 Pro 成員：Pro 帳號同樣切得到 Fable，「`/model fable` 能切換」不代表含在方案內，所有用到 `fable` 的環節（`speckit-specify`／`speckit-plan`、`spec-challenger` 代位、真人 grill 的 `/model fable`）都會以 usage credits 計費；開工前要照步驟 7 寫入 CLAUDE.md 的段落做個人設定並驗證。

規則：任何會覆蓋既有檔案的動作先備份並徵詢；本 command 可重複執行（例如 spec-kit 升級後重跑以恢復 frontmatter patch，或訂閱方案變更後重跑步驟 5 換配置）。
