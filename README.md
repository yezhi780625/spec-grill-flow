# spec-grill-flow

團隊 AI 交辦任務流程 plugin：**Spec-Kit**（文件骨架）＋ **grill**（需求錘鍊）＋ **Superpowers**（實作紀律），含分流快速通道與合併後 retro。

## 流程總覽

```
分流 ─┬─ 完整通道：[P1 定義] → [P2 錘鍊] → [P3 規劃] → [P4 實作] → [P5 驗收] → [P6 Retro]
      ├─ 輕量通道：[P1+P2 寫入即錘鍊] ──→ [P3 簡化] → [P4 實作] → [P5 驗收] → [P6 Retro]
      └─ 快速通道：──────────────────────────────→ [P4 實作] → [P5 驗收] → [P6 Retro]
```

Phase 0 宣告專案有無真人協作者，何時用人由分流決定——完整通道真人 grill（兼知識擴散）、輕量通道與無真人時由 fresh-context agent 代位。內建退回熔斷與數據量驅動的 retro 檢視節奏。

核心原則：spec 文件是唯一真相源。完整說明見 [skills/team-workflow/full-workflow.md](skills/team-workflow/full-workflow.md)。

## 安裝

```
/plugin marketplace add yezhi780625/spec-grill-flow
/plugin install spec-grill-flow@spec-grill-flow-marketplace
```

安裝時會自動帶入依賴的 Superpowers plugin。之後在每個要使用的專案跑一次：

```
/spec-grill-flow:setup
```

必要環境：Node.js、Python 3.10+ 與 uv（spec-kit 用）。建議配置：Claude Code v2.1.219+ 與 Opus 5 / Fable 5（setup 會先徵詢再套用模型設定，無對應存取權也能使用本流程）。

## 內容

- `skills/team-workflow/` — 流程 skill（agent 讀）＋ full-workflow.md（人讀：負責人、Phase 0、ECC 引入條件）
- `skills/grill-me/` — spec 錘鍊訪談（vendored from [mattpocock-skills](https://github.com/mattpocock)，MIT）
- `commands/setup.md` — 一次性專案初始化
- `templates/` — constitution、CLAUDE.md 鐵律段落、PR template、retro-log

## License

MIT（見 [LICENSE](LICENSE)）。grill-me skill 源自 mattpocock-skills（MIT），見檔內標注。
