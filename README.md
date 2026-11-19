# spec-grill-flow

團隊 AI 交辦任務流程 plugin：**steelman**（目標確認）＋ **Spec-Kit**（文件骨架）＋ **grill**（需求錘鍊）＋ **fresh-context subagent**（TDD 實作），含分流快速通道。不依賴其他 plugin。

## 流程總覽

```
提案（feature request／PRD／自發 feature）──[目標確認 steelman：挖目標→PM 確認]──┐ （撤回／擋下：不進流程）
回報與無新行為任務（bug fix／typo／重構…）───────────────────────────────────────┤
                                                                                 ▼
                                             分流 ─┬─ 完整通道：[P1 定義] → [P2 錘鍊] → [P3 規劃] → [P4 實作] → [P5 驗收]
                                                   ├─ 輕量通道：[P1+P2 寫入即錘鍊] ──→ [P3 簡化] → [P4 實作] → [P5 驗收]
                                                   └─ 快速通道：──────────────────────────────→ [P4 實作] → [P5 驗收]
```

Phase 0 宣告專案有無真人協作者，何時用人由分流決定——完整通道真人 grill（兼知識擴散）、輕量通道與無真人時由 fresh-context agent 代位。實作由主 agent 派 fresh-context subagent 執行，model 與 effort 由主 agent 判斷。內建退回熔斷。

核心原則：spec 文件是唯一真相源。完整說明見 [skills/team-workflow/full-workflow.md](skills/team-workflow/full-workflow.md)。

## 安裝

```
/plugin marketplace add yezhi780625/spec-grill-flow
/plugin install spec-grill-flow@spec-grill-flow-marketplace
```

之後在每個要使用的專案跑一次：

```
/spec-grill-flow:setup
```

必要環境：Node.js、Python 3.11+ 與 uv（spec-kit ≥1.0——指令為 `/speckit-*` skill 形式；2026-11-19 以 1.4 實測，較早的 1.x 版本未測）。本流程不指定模型與 effort，由使用者自行選擇；既有專案（spec-kit 0.x，或跑過 0.7.0 之前任何版本的 setup）升級到 0.7.0 後，請跑一次 `/spec-grill-flow:migrate`：它會清理舊版寫入的模型設定、把 spec-kit 專案檔升到 1.x、同步 CLAUDE.md 段落與 PR template，並詢問如何處理已停用的 retro-log 與 constitution 舊條文。0.7.0 起不再依賴 Superpowers；已安裝的人可以留著自用：CLAUDE.md 鐵律寫明它的規劃與執行類 skill 不在本流程使用（2026-11-19 以 Superpowers 7.0.2 實測 4 次，都沒有被載入）。

## 內容

- `skills/team-workflow/` — 流程 skill（agent 讀）＋ full-workflow.md（人讀：負責人、Phase 0、熔斷細節）
- `skills/steelman-requirement/` — 分流前的目標確認站：提案先 steelman（挖目標、最強論證、確認句），目標確認才寫 spec；bug 回報免
- `skills/grill-me/` — spec 錘鍊訪談（vendored from [mattpocock-skills](https://github.com/mattpocock/skills)，MIT）
- `skills/diagnosing-bugs/` — Phase 4 的除錯法：先重現、找到根因才修（vendored from mattpocock-skills，MIT）
- `skills/migrate/` — 既有專案跨大版本的遷移（spec-kit 0.x 到 1.x、清理舊版產物）
- `commands/setup.md` — 一次性專案初始化
- `templates/` — constitution、CLAUDE.md 鐵律段落、PR template

## License

MIT（見 [LICENSE](LICENSE)）。grill-me 與 diagnosing-bugs skill 源自 mattpocock-skills（MIT），見檔內標注。
