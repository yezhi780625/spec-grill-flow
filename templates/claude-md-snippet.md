# CLAUDE.md 追加段落（貼進專案的 CLAUDE.md）

## 交辦流程鐵律

- Spec、plan、tasks 一律使用 `/speckit-*` 指令產出（spec-kit ≥0.8.10 的 skill 形式，連字號；不使用 /brainstorm、/write-plan）。
- `specs/` 下的 spec 文件是唯一真相源；需求變更先改 spec 並通過 grill 檢核，再繼續實作。
- 實作採 TDD：測試先寫、先失敗，才寫實作（豁免類別與程序見 constitution I，禁止空測試充數）。
- 完整流程與驗收清單見 team-workflow skill。**規則書不在本 repo 內**——隨 spec-grill-flow plugin 散佈，未安裝就查不到。先跑：

  ```
  /plugin marketplace add yezhi780625/spec-grill-flow
  /plugin install spec-grill-flow@spec-grill-flow-marketplace
  ```

  裝完 plugin 即可開工——setup 產出的檔案（constitution、PR template、retro-log 等）已在版控內，全員共享；只有本專案尚未初始化（無 `.specify/`）時才需跑一次 `/spec-grill-flow:setup`。刻意不把規則書複製進本 repo：那會與上游形成第二份會走鐘的複本，違反 constitution IV（單一真相源）。
