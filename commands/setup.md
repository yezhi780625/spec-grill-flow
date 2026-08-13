---
description: 一次性初始化：在當前專案部署 spec-grill-flow 工作流（spec-kit、constitution、CLAUDE.md、PR template、retro-log、模型設定）。
disable-model-invocation: true
---

在當前專案執行 spec-grill-flow 的 Phase 0 初始化。逐步執行，每步先偵測現況、冪等處理（已存在則跳過或詢問），全部完成後輸出總結表。

1. **前置檢查**：
   - 確認 `specify` CLI 可用（`uvx --from git+https://github.com/github/spec-kit.git specify check` 或已安裝的 specify）。不可用 → 停下，給使用者安裝指令，等待完成後再繼續。
   - 確認 Superpowers plugin 已載入（檢查 /brainstorm 等指令是否存在）。未載入 → 提示 plugin 依賴可能未解析，給手動安裝指令。
2. **spec-kit 初始化**：repo 中不存在 `.specify/` → 引導執行 `specify init --here`；已存在則跳過。
3. **部署 constitution**：`.specify/memory/constitution.md` 若為預設模板或不存在 → 以本 plugin `templates/constitution.md` 為底，詢問使用者填入 `<角括號>` 參數後寫入；若已有客製化內容 → 先備份為 `constitution-backup.md` 再詢問是否合併。
4. **模型設定**：在 `.claude/settings.json` 寫入（合併，勿覆蓋既有鍵）`"model": "opus"`, `"effortLevel": "high"`。對 `.claude/commands/speckit-specify.md`、`speckit-plan.md`、`speckit-tasks.md` 的 frontmatter 加入 `model: fable` 與 `effort: medium`（已存在則跳過）。
5. **CLAUDE.md**：將 `templates/claude-md-snippet.md` 的「交辦流程鐵律」段落 append 到專案 CLAUDE.md（已含該段落則跳過）。
6. **PR template**：將 `templates/pr-template.md` 複製到 `.github/pull_request_template.md`（已存在則詢問合併方式）。
7. **Retro log**：建立 `specs/retro-log.md`（以 `templates/retro-log.md` 為底，已存在則跳過）。
8. **驗證與總結**：列出每一步的結果（新建／跳過／備份後覆蓋），提示使用者 commit 這些變更，並提醒團隊成員各自跑一次 `/model fable` 確認帳號有 Fable 存取權。

規則：任何會覆蓋既有檔案的動作先備份並徵詢；本 command 可重複執行（例如 spec-kit 升級後重跑以恢復 frontmatter patch）。
