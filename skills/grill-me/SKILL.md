---
name: grill-me
description: 對一份計畫、spec 或設計進行不留情的逼問訪談，直到達成共同理解。本流程中主要用於 Phase 2：對著 spec 文件錘鍊。
---

<!-- Vendored from mattpocock-skills (grill-me + grilling), MIT.
     上游同步：GitHub Actions（.github/workflows/check-upstream-skill.yml）每週比對 .upstream.json 釘住的
     commit，有差異時由 Claude 自動改寫並開 PR（失敗則退回開 issue）；由 Phase 0 指定的 owner review 該 PR。 -->

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled — the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Each round the user answers reshapes the tree — settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it — don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report — ask the rest of the frontier now. The _decisions_ are the user's — put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.

## 本流程的附加規則（team-workflow Phase 2）

- **grill 執行者不得是 spec 作者**，依通道決定：完整通道且專案有真人時由非作者成員主持（兼知識擴散）；其餘（輕量通道或無真人）由 fresh-context 的獨立 subagent 執行——受審對象是 spec 文件本身，不帶產生該 spec 的對話脈絡（repo 與事實查證照常可用），prompt 明確指定挑錯立場。
- 逼問對象是 spec 文件時，每一輪定案的答案**立即寫回 spec 並 commit**，再進下一輪——git log 即訪談紀錄。
- 作者可駁回追問，但被駁回的追問記入 spec 的「未解事項」段——不得刪除，Phase 5 reviewer 會檢視。
- 追問至少涵蓋四個維度：驗收標準可量測性、edge cases 與失敗路徑、需求矛盾、「不做什麼」。
- 產出只寫回 spec 本身，不另產 ADR、glossary 或其他文件。
- 結束前對照 spec 頂部的 Grill 檢核清單，四項可勾才算完成。
