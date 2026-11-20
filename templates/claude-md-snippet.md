# CLAUDE.md 追加段落（貼進專案的 CLAUDE.md）

## 交辦流程鐵律

- Spec、plan、tasks 一律使用 `/speckit-*` 指令產出（spec-kit ≥1.0 的 skill 形式，連字號）。
- `specs/` 下的 spec 文件是唯一真相源；需求變更先改 spec 並通過 grill 檢核，再繼續實作。
- 實作由主 agent 派 fresh-context subagent 執行，小修改也一樣（使用者明確要求直接改時除外）；model 與 effort 由主 agent 判斷。
- 一個 agent 一個 git worktree。
- 實作採 TDD：測試先寫、先失敗，才寫實作。新行為的測試 commit 排在實作的 commit 之前；事後新增或修改的斷言，要改壞對應行為、確認它轉紅，並把結果寫進 PR 描述（豁免類別與程序見 constitution 的 Test-First 原則，禁止空測試充數）。
- 完成的任務，在完成它的那個 PR 裡勾選 tasks.md。開 PR（或 MR）時以專案的範本作為描述骨架。
- 規劃與實作只用上述方式。其他 plugin 的規劃與執行類 skill（例如 Superpowers 的 `brainstorming`、`writing-plans`、`subagent-driven-development`、`executing-plans`）不在本流程使用。
- 完整流程與驗收清單見 team-workflow skill。**規則書不在本 repo 內**——隨 spec-grill-flow plugin 散佈，未安裝就查不到。先跑：

  ```
  /plugin marketplace add yezhi780625/spec-grill-flow
  /plugin install spec-grill-flow@spec-grill-flow-marketplace
  ```

  裝完 plugin 即可開工——setup 產出的檔案（constitution、PR template 等）已在版控內，全員共享；只有本專案尚未初始化（無 `.specify/`）時才需跑一次 `/spec-grill-flow:setup`；例外：跑過 0.7.0 之前任何版本 setup 的專案，升級到 0.7.0 後跑一次 `/spec-grill-flow:migrate`，以清理舊版產物並把 spec-kit 升到 1.x。刻意不把規則書複製進本 repo：那會與上游形成第二份會走鐘的複本，違反 constitution 的「Spec 為唯一真相源」原則。
