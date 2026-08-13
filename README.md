# spec-grill-flow

團隊 AI 交辦任務流程 plugin：**Spec-Kit**（文件骨架）＋ **grill**（需求錘鍊）＋ **Superpowers**（實作紀律），含分流快速通道與合併後 retro。

## 流程總覽

```
分流 ─┬─ 完整通道：定義 → 錘鍊 → 規劃 → 實作 → 驗收 → Retro
      └─ 快速通道：────────────────→ 實作 → 驗收 → Retro
```

核心原則：spec 文件是唯一真相源。完整說明見 [skills/team-workflow/full-workflow.md](skills/team-workflow/full-workflow.md)。

## 安裝

```
/plugin marketplace add YOUR_GITHUB_USER/spec-grill-flow
/plugin install spec-grill-flow@spec-grill-flow-marketplace
```

安裝時會自動帶入依賴的 Superpowers plugin。之後在每個要使用的專案跑一次：

```
/spec-grill-flow:setup
```

前置需求：Claude Code v2.1.219+（Opus 5 / Fable 5）、Node.js、Python 3.10+ 與 uv（spec-kit 用）。

## 內容

- `skills/team-workflow/` — 流程 skill（agent 讀）＋ full-workflow.md（人讀：負責人、Phase 0、ECC 引入條件）
- `skills/grill-me/` — spec 錘鍊訪談（vendored from [mattpocock-skills](https://github.com/mattpocock)，MIT）
- `commands/setup.md` — 一次性專案初始化
- `templates/` — constitution、CLAUDE.md 鐵律段落、PR template、retro-log

## License

MIT。grill-me skill 源自 mattpocock-skills（MIT），見檔內標注。
