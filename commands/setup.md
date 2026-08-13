---
description: 一次性初始化：在當前專案部署 spec-grill-flow 工作流（spec-kit、constitution、人力宣告與 owner、CLAUDE.md、PR template、retro-log、模型設定）。
disable-model-invocation: true
---

在當前專案執行 spec-grill-flow 的 Phase 0 初始化。逐步執行，每步先偵測現況、冪等處理（已存在則跳過或詢問），全部完成後輸出總結表。

1. **前置檢查**：
   - 確認 `specify` CLI 可用（`uvx --from git+https://github.com/github/spec-kit.git specify check` 或已安裝的 specify）。不可用 → 給使用者安裝指令（含 uv 本身的安裝方式），可先跳過此步完成其餘步驟，最後在總結表標注待補。
   - 確認 Superpowers plugin 已載入（檢查 /brainstorm 等指令是否存在）。未載入 → 提示 plugin 依賴可能未解析，給手動安裝指令，同樣可先跳過。
2. **spec-kit 初始化**：repo 中不存在 `.specify/` → 引導執行 `specify init --here`；已存在則跳過。
3. **部署 constitution**：`.specify/memory/constitution.md` 若為預設模板或不存在 → 以本 plugin `templates/constitution.md` 為底，詢問使用者填入 `<角括號>` 參數後寫入；若已有客製化內容 → 先備份為 `constitution-backup.md` 再詢問是否合併。
4. **宣告人力與 owner**：詢問專案有無真人協作者（用不用人由分流決定——完整通道真人 grill、其餘 AI 代位，見 team-workflow），寫入 constitution 的 Governance 段，並填齊三個 owner（constitution 修訂核准人、grill-me 上游同步、ECC agent 若引入）。無真人時三者皆為使用者本人。**不得留 placeholder**——這是 retro 檢視機制的觸發依據。
5. **模型設定（建議配置，先徵詢再寫入）**：設計理由——spec 與規劃環節（speckit 三指令）用最強的 Fable（能力高於 Opus、單價約兩倍、速度較慢），因為需求想錯的成本高於實作瑕疵，且此環節 token 量小、以 `effort: medium` 控制花費；主對話量大，用 Opus。向使用者說明時勿描述為「快且便宜」。詢問使用者是否套用建議模型配置；同意才在 `.claude/settings.json` 寫入（合併，勿覆蓋既有鍵）`"model": "opus"`, `"effortLevel": "high"`，並對 `.claude/commands/speckit.specify.md`、`speckit.plan.md`、`speckit.tasks.md` 的 frontmatter 加入 `model: fable` 與 `effort: medium`（已存在則跳過）。使用者婉拒或無對應模型存取權 → 跳過，流程照常可用。
6. **CLAUDE.md**：將 `templates/claude-md-snippet.md` 的「交辦流程鐵律」段落 append 到專案 CLAUDE.md（已含該段落則跳過）。
7. **PR template**：將 `templates/pr-template.md` 複製到 `.github/pull_request_template.md`（已存在則詢問合併方式）。
8. **Retro log**：建立 `specs/retro-log.md`（以 `templates/retro-log.md` 為底，已存在則跳過）。
9. **驗證與總結**：列出每一步的結果（新建／跳過／待補／備份後覆蓋），提示使用者 commit 這些變更；若套用了模型設定，建議團隊成員各自跑一次 `/model fable` 確認帳號有 Fable 存取權。

規則：任何會覆蓋既有檔案的動作先備份並徵詢；本 command 可重複執行（例如 spec-kit 升級後重跑以恢復 frontmatter patch）。
