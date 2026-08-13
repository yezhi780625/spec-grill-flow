# CLAUDE.md 追加段落（貼進專案的 CLAUDE.md）

## 交辦流程鐵律

- Spec、plan、tasks 一律使用 `/speckit.*` 指令產出（不使用 /brainstorm、/write-plan）。
- `specs/` 下的 spec 文件是唯一真相源；需求變更先改 spec 並通過 grill 檢核，再繼續實作。
- 實作採 TDD：測試先寫、先失敗，才寫實作（豁免類別與程序見 constitution I，禁止空測試充數）。
- 完整流程與驗收清單見 team-workflow skill。
