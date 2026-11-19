# AI 交辦任務 Workflow（團隊版）

**工具鏈**：steelman-requirement（目標確認）＋ Spec-Kit（文件骨架與一致性檢查）＋ grill-me（需求錘鍊）＋ diagnosing-bugs（除錯）。實作紀律由本流程的 Phase 4 與 constitution 規定，不依賴其他 plugin。

> 本文件是流程的唯一長版真相源；[SKILL.md](SKILL.md) 是 agent 讀的濃縮版，兩者不一致時以本文件為準。

---

## 總覽

```
提案（feature request／PRD／自發 feature）──[目標確認 steelman：挖目標→PM 確認]──┐ （撤回／擋下：不進流程）
回報與無新行為任務（bug fix／typo／重構…）───────────────────────────────────────┤
                                                                                 ▼
                                             分流 ─┬─ 完整通道：[P1 定義] → [P2 錘鍊] → [P3 規劃] → [P4 實作] → [P5 驗收]
                                                   │             speckit     grill-me    speckit     subagent     spec 對照
                                                   ├─ 輕量通道：[P1+P2 寫入即錘鍊] → [P3 簡化] → [P4 實作] → [P5 驗收]
                                                   └─ 快速通道：──────────────────────────────→ [P4 實作] → [P5 驗收]
```

核心原則：**Spec-Kit 是唯一文件真相源**。所有需求、計畫、任務都活在 repo 的 spec 文件裡；grill 和實作過程的所有結論，最終都必須寫回 spec，否則視為不存在。

---

## 目標確認：steelman（分流之前，僅限提案）

**負責人**：需求收受者（接到需求的工程師），PM 作答。PM 通常不在開發環境——step 4 的確認問句由 RD 帶去向 PM 核對後**代為轉達**；代答是轉達 PM 的回覆，不是代 PM 拍板。目標寫入 spec 時標注確認方式（誰、何時、什麼管道）。PM 不回時 RD 照常催促——開發本來就 block 在 PM 回覆上，這不是本站新增的等待。

收到提案類需求（feature request／PRD／方案性需求）時，先用 `steelman-requirement` 過一輪：挖出需求背後的目標、把需求重建成最強版本、再給技術與產品意見，最後以確認問句向 PM 核對目標。

**本站不裁決「該不該做」**——那是產品決策，RD 通常沒有否決權，硬擋語義只會被繞過。它保證的是：**目標確認完成，才寫 spec**。真實機制是說服而非否決權——一份好的 steelman 加上對最強版本的疑慮，常讓 PM 自行收斂或撤回需求。唯一保留擋下語義的是內部自發 feature：RD 對自己團隊的提案有裁量權。

**本站攔提案，不攔回報**。判別式：對方期望的行為，spec（或現有明確行為）已描述了嗎？

- **已描述** → 這是 bug 回報：目標自明（回到既定行為），「該不該做」無疑問，跑 steelman 是純儀式——免跑直接分流，正好落在快速通道對 bug fix 的定義（「回到 spec 已描述的行為」），兩者無縫。豁免真回報同時是在保護本站本身：每次都跑的儀式很快會被整個跳過，連該跑的提案一起。
- **未描述** → 「這是 bug」其實是包裝成回報的需求變更（PM 認為應該如此，但 spec 從未承諾）——這正是最需要挖目標的偽裝提案，照跑。
- **拿不準** → 跑。誤跑的成本是一則回覆，漏跑的成本是一個錯 feature，與分流「猶豫升級」同哲學。

內部任務中，回報與無新行為者（typo、重構、回到既定行為的修正）同樣免跑，直接分流。**內部自發的新 feature 同為提案，照跑**——提案要有依據，而「依據」正是 step 1 要產出的目標。差別只在執行者：自己 steelman 自己的提案，與自己 grill 自己的 spec 是同一種盲點，所以執行者不得是提案者本人——有真人由同事執行，無真人由 fresh-context subagent 代位（規則見「人力宣告」），提案者作答。

- **目標已確認** → 進入分流。定案的「需求背後的目標」段落連同需求原文一起帶入：完整通道作為 Phase 1 `/speckit-specify` 的輸入，成為 spec 動機（motivation）段落的雛形；輕量通道隨該條驗收標準的增修一併寫入 spec。**未被採納的疑慮不隨 PM 拍板蒸發**——以「已知取捨」寫入 spec 的「未解事項」段，Phase 5 合併前逐項定案的既有機制會回看它。
- **目標未獲確認** → 不寫 spec、不進分流。**未經 PM 確認的目標不得寫進 spec**——steelman 重建的目標有腦補風險，寫進唯一真相源前必須先核對。
- **PM 撤回或改寫需求，或內部提案被擋下** → 不進流程；改寫後的新需求重新過一次本站。

**與 grill 的分工**：steelman 確立「為什麼做」（目標與論證），grill（Phase 2）錘「怎麼做才驗收得了」（驗收標準可量測性、edge cases 與失敗路徑、需求矛盾、「不做什麼」）。維度不同，過了 steelman 不代表免 grill，兩者不可互抵。

---

## 人力宣告（Phase 0 宣告一次）

Phase 0 只宣告一件事：**專案有沒有可用的真人協作者**。用不用人不是專案級開關，由分流決定——同一個專案裡，小需求個人＋AI 就地解決，大需求自動走到真人面前：

- **快速通道**：不 grill。
- **輕量通道**：AI 代位 grill——單條驗收標準的誤解成本低，fresh-context agent 的追問品質足夠。
- **完整通道**：有真人 → **真人 grill**（主持人不得是 spec 作者）；無真人 → AI 代位。

升級規則自動繼承：任務從輕量升到完整通道時，grill 執行者一併升級。不新增任何自評旋鈕——判斷猶豫時升級通道，就升級了 grill。

**真人 grill 有兩個功能，這條規則靠第二個立足**：(1) 獨立挑錯——這部分 AI 代位做得不差；(2) **知識擴散**——grill 過這份 spec 的同事，從此知道這個 feature 存在、為什麼做、邊界在哪，這是 bus factor 與後續 review 品質的來源，AI 無法替代。所以「完整通道找真人」不會被「agent 也審得很好啊」侵蝕：審得好只覆蓋功能 (1)。

**AI 代位規則**（適用於所有由 agent 擔任「非本人」角色的場合）：fresh-context 的獨立 subagent。關鍵不是換個名字，而是換個脈絡——**不得共享產生受審產物的對話脈絡**（撰寫 spec、實作程式的過程對話），但**可自由讀取 repo**：codebase 認識是審查的合法輸入，該擋的只有附和傾向，不是知識。prompt 明確指定挑錯立場。參與過推理的 agent 會順著自己的假設附和，等於沒審。

**模型與 effort（不由本流程指定）**：主對話的模型交給使用者選擇——用 `/model`、`/effort` 切換，個人偏好寫在 `.claude/settings.local.json`，不寫進團隊共用的 `settings.json` 或 skill frontmatter。理由：模型之間的強弱排序幾個月就會變，寫死在流程裡的建議很快過期。流程只規定兩件事：

- **派出 subagent（代位 grill、Phase 4 實作、代位驗收）時，model 與 effort 由主 agent 依任務判斷決定**。不指定時，subagent 沿用主對話的設定（未設定環境變數 `CLAUDE_CODE_SUBAGENT_MODEL`、也沒有使用自帶 `model` 的 subagent 定義時）。依據：2026-11-19 以 v2.1.328 實測 97 次派工，派工工具有 model 與 effort 兩個參數；主 agent 有 41 次指定 model、從未指定 effort，其餘都沿用主對話的模型。
- **選擇範圍是使用者訂閱方案內含的模型**。會在方案額度外另外計費的模型，使用者明確要求才用——agent 對模型強弱的認知來自訓練資料，會過期，不足以當成讓使用者多付費的理由。

要為特定環節指定模型或 effort，走一般的流程修訂：PR 修改本文件，附上證據（例如代位 grill 的任務在 PR 驗收反覆出現「AI 做出來的不是想要的」），並寫明對方案外另計費或無存取權成員的處理方式。

---

## 分流（每個任務的第一步）

依**任務性質**選通道——不需要先精讀 spec 預測差異。判斷猶豫時，一律往下選更完整的通道。

### 快速通道

任務屬於以下類型之一，且不引入新依賴、不新增抽象層：

- typo／文案修正
- 依賴升級
- bug fix（使程式碼回到 spec 已描述的行為）
- 不改外部行為的重構

「變更範圍約 3 個檔案」是**絆線而非資格門檻**：事前不強制精算；PR diff 實際超過 3 檔時，由 Phase 5 reviewer 判斷是否應升級通道，而非讓任務自動作廢。

流程：直接進 Phase 4（TDD 照常強制：先寫重現問題的失敗測試）→ Phase 5（免 spec 逐條對照，確認「失敗測試→修正→綠燈」證據鏈）。

### 輕量通道

需要動 spec，但只**新增或修改 1 條驗收標準**，且不引入新依賴、不新增抽象層。（「1 條」是啟發式而非客觀量尺——兩條簡單標準可能比一條複雜標準輕鬆；誤判由升級規則兜底。）

流程：

1. **P1＋P2 合併（寫入即錘鍊）**：直接在 spec 增修該條標準並 commit；立刻只針對這條標準跑 grill 四維度各至少一問（單條標準數分鐘可完），修正再 commit。grill 由 AI 代位執行（規則見「人力宣告」）。
2. **P3 簡化**：在 tasks 補一條任務描述（含對應測試任務），免完整 plan。
3. **P4–P5 照常**。Phase 5 的逐條對照僅限本次增修的標準與其 edge cases。

### 完整通道

其餘一切：新 feature、牽動多條驗收標準、引入新依賴或抽象層、或任何拿不準的情況。走完整 Phase 1–5。

### 升級規則（單向）

任何通道進行中，一旦發現實際範圍超出該通道資格——立即停手，升級到對應通道（快速→輕量或完整；輕量→完整）。

**已寫的測試與程式碼不作廢**：全部帶入升級後的 Phase 4 繼續使用，只回頭補做 Phase 1–3 的 spec 與規劃工作。升級的成本是補文件，不是重來——低估通道等級不會讓你損失實作進度，所以不必為了保住進度而硬撐在原通道。

判斷猶豫時，一律升級。此規則單向：不得反向降級。

---

## Phase 0：專案初始化（一次性）

新專案由 `/spec-grill-flow:setup` 執行以下步驟。既有專案（spec-kit 0.x，或跑過本 plugin 0.7.0 之前的 setup）改跑 `/spec-grill-flow:migrate`。

1. `specify init --here --force --integration claude` 建立專案骨架（spec-kit ≥1.0；兩個旗標的理由見 `/spec-grill-flow:setup` 步驟 2）。
2. `/speckit-constitution` 定義專案原則，至少包含：
   - 核心函式必須附單元測試（測試先於實作，豁免條款見 constitution I）
   - 每條驗收標準必須可量測（可自動驗證優先）
   - Spec 未通過 grill 檢核不得進入 plan 階段
3. 在 CLAUDE.md 寫入「交辦流程鐵律」段落（`templates/claude-md-snippet.md`）：spec、plan、tasks 一律用 `/speckit-*`；實作依 Phase 4；其他 plugin 的規劃與執行類 skill 不在本流程使用。CLAUDE.md 每次對話都會載入，所以最容易被跳過的規則放在這裡。
4. `grill-me` 與 `diagnosing-bugs` 已 vendored 於本 plugin（`skills/grill-me/`、`skills/diagnosing-bugs/`，各附 `.upstream.json`，plugin repo 的 CI 每週比對上游、有差異自動開 PR），安裝 plugin 即有。**勿再手動從 mattpocock-skills 複製進專案的 `.claude/skills/`**——那會形成第二份不受 CI 同步保護的複本，違反單一真相源；也**不安裝 mattpocock-skills 整包**（to-spec、to-tickets、grill-with-docs、implement 等與 spec-kit 及本流程職責重疊）。
5. 在 PR template 加入 spec 驗收清單區塊（見 Phase 5）。
6. **宣告人力與核准人**：在 constitution 的 Governance 段寫明專案有無真人協作者（用不用人由分流決定，見「人力宣告」），並填上 constitution 修訂核准人。無真人時為本人。

---

## Phase 1：需求定義（Spec-Kit）

**負責人**：需求發起者（工程師或 PM）

1. 執行 `/speckit-specify`，把原始需求丟進去，產出 spec 草稿。經目標確認的任務，把 steelman 定案的「需求背後的目標」段落一併作為輸入——它是 spec 動機段落的雛形，「為什麼做」從第一天就寫在 spec 裡。
2. 從預設分支建立 feature branch，名稱與 `/speckit-specify` 剛建立的 feature 目錄相同：`git switch -c <feature 目錄名> <預設分支>`（例如目錄是 `specs/001-duration-parsing`，分支就是 `001-duration-parsing`；還沒 commit 的草稿會跟著過去）。已經在同名分支上則跳過。spec-kit 1.x 的 `/speckit-specify` 自己不建分支；由流程建立，分支名才能和 feature 目錄對得起來。
3. 草稿 commit 進這個分支——**先有文件，再有討論**。

**speckit 指令作用在哪個 feature**：spec-kit 1.x 只看 `.specify/feature.json`（各人本機的檔案，不進版控），不看目前的 git 分支。`/speckit-specify` 會寫入它；但切換分支、接手別人的分支、reviewer 在自己的 checkout 重跑、或輕量通道修改既有 spec 時，它可能不存在或指向別的 feature——不存在時指令報錯，過期時指令會默默作用在另一個 feature 上。所以在這些情況下執行 `/speckit-plan`、`/speckit-tasks`、`/speckit-analyze`、`/speckit-converge` 之前，先確認它的內容是 `{"feature_directory":"specs/<本次的 feature 目錄>"}`，不是就改寫。完整通道的分支與 feature 目錄同名，所以就是 `specs/<目前分支名>`；輕量通道是被修改的那份 spec 所在的目錄。

> 此時的 spec 只是「格式化的初稿」，品質未經檢驗，不得直接進入 Phase 3。

---

## Phase 2：需求錘鍊（grill-me 對著 spec 打）

**負責人**：**非 spec 作者**——與 Phase 5「非實作者 review」同構。需求誤解的成本高於實作瑕疵，自己 grill 自己寫的需求是流程最早、也最貴的盲點。執行者依通道決定（見「人力宣告」）：

- **完整通道且專案有真人**：由非作者成員主持 grill，spec 作者作答——同時完成知識擴散。
- **其餘情況**（輕量通道，或無真人可用）：由 fresh-context 的獨立 subagent 執行——受審對象是 spec 文件本身，不帶 Phase 1 的對話脈絡（repo 與事實查證照常可用），prompt 明確指定 adversarial 立場；作者本人作答。代位 subagent 無法直接向作者提問：每一輪的問題由主對話轉給作者，作答後把答案交回**同一個** subagent 續問（保留訪談脈絡），不要每輪另開新的。主對話帶著 Phase 1 的脈絡，所以**只能原文轉送**：不得改寫問題、加註說明或代作者作答。每輪由代位 subagent commit，message 以 `grill:` 開頭，含它提出的問題原文與主對話原文轉交的作者答案；該輪沒有改 spec 時用空 commit（`git commit --allow-empty`）記錄，供 Phase 5 查核。

真人 grill 時，主持人可主動補充作者不掌握的事實（既有行為、歷史決策、其他系統的限制），寫入 spec 並標注來源；但取捨與拍板仍屬作者。**若一次 grill 出現三項以上此類補充**，代表該領域的事實只存在同事腦中：開一個 issue 把這些知識補進專案文件。補上之前，同領域的任務分流偏向完整通道（AI 代位會漏掉這些事實）；補上之後就不必每次消耗真人。

1. 開啟 `grill-me`，指定目標為該份 spec 文件，接受逼問。追問焦點至少涵蓋：
   - 每條驗收標準怎麼量測？由誰／什麼工具判定通過？
   - Edge cases 列了嗎？失敗路徑呢？
   - 需求之間有沒有互相矛盾？
   - 範圍邊界：明確「不做什麼」了嗎？
2. **每一輪修正直接改 spec 文件並 commit**——git log 就是訪談紀錄，reviewer 之後看 diff 就知道這份 spec 被錘過哪裡。
3. 作者有權駁回或跳過任何追問，但**被駁回的追問由 grill 執行者記入 spec 的「未解事項」段——不得刪除，Phase 5 reviewer 會檢視**。跳過從無聲無息變成留痕到驗收；這是 AI 代位 grill 裡「同事皺眉頭」的替代品（AI 代位的攔截力天生弱於真人，此留痕是主要補償機制）。
4. 完成後在 spec 文件頂部勾選 grill 檢核清單：

```markdown
## Grill 檢核
- [ ] 所有驗收標準可量測
- [ ] Edge cases 與失敗路徑已列出
- [ ] 無互相矛盾的需求
- [ ] 「不做什麼」已明確
```

**規則**：grill 的產出只能寫回 spec 本身，不另產 ADR／glossary／其他文件。

---

## Phase 3：規劃與拆解（Spec-Kit）

**負責人**：實作者

1. `/speckit-plan` 產出技術計畫（架構選擇、依賴、資料流）。
2. `/speckit-tasks` 拆解為可交辦的任務清單。
3. `/speckit-analyze`（完整通道；輕量通道免完整 plan，不適用）檢查 spec、plan、tasks 與 constitution 是否一致。它是唯讀的，只產出報告；CRITICAL 與 HIGH 的發現要回到源頭修正後重跑，直到沒有這兩級的發現：
   - 問題在 plan 或 tasks（例如某條驗收標準沒有對應的測試任務）→ 在本階段修正。
   - 問題在 spec → **退回 Phase 2 補 grill**。
   - 不同意某項發現 → 不必為它重跑：把該項與理由記入 spec「未解事項」，由 Phase 5 定案。
   - 這是規劃者的自我檢查。結果寫進 plan 與 tasks 的 commit message（例如 `analyze: 0 CRITICAL, 0 HIGH`，有記入未解事項的另外註明），reviewer 有疑慮時可自行重跑。
4. Plan 與 tasks 一併 commit。若 plan 或 analyze 發現 spec 有洞 → **退回 Phase 2 補 grill**，改完 spec 再回來（退回計數見「退回與熔斷」）。

`/speckit-tasks` 只在 spec 或 constitution 要求時才產生測試任務。本流程的 constitution I 有這項要求，所以不必另外指定；專案若改寫或拿掉了 constitution I，就要在呼叫 `/speckit-tasks` 時明確要求 TDD。

---

## Phase 4：實作（fresh-context subagent＋TDD）

**負責人**：實作者＋AI agent

1. **派工**：主 agent 依 tasks.md 的順序，把任務交給 **fresh-context subagent** 實作。主 agent 負責派工、核對回報與整合；實作的程式碼由 subagent 寫。主 agent 載入 `/speckit-implement` 當入口也可以，派工、commit 與勾選仍依本階段的規則（2026-11-19 實測 5 次，載入後都照樣派 subagent）。
   - 一次交辦的範圍由主 agent 決定：單一任務，或 tasks.md 的一個階段／一個 user story。會改到同一個檔案的任務依序派。
   - model 與 effort 由主 agent 判斷（見「模型與 effort」）。
   - 交辦內容用路徑指向 spec、plan、tasks.md 與 constitution，列出這次的任務編號，並附上本階段第 2 到 5 點的規則——subagent 看不到主對話，也不會自己載入本 skill。
   - subagent 回報後，主 agent 自己執行一次完整測試，確認綠燈才派下一批。
   - 理由：fresh context 的實作者只看得到文件，spec 與 tasks 沒寫清楚的地方會在這裡暴露，而不是被主對話的記憶補上；主對話的 context 留給協調與判斷。
2. **TDD（constitution I）**：每個行為先寫測試，執行並看到它失敗，才寫實作。**測試的 commit 排在對應實作的 commit 之前**，commit 訊息帶任務編號——Phase 5 靠這個順序驗證測試先行，跳過即打回。豁免類別（純 UI 樣式、一次性 migration、無邏輯膠水碼）走 constitution I 的豁免程序——附替代驗證方式，禁止空測試充數。
3. **完成一項勾一項**：任務完成且測試綠燈後，把 tasks.md 對應的 `- [ ]` 改成 `- [x]` 並 commit。tasks.md 是進度的唯一紀錄，主 agent 與之後接手的人都靠它判斷做到哪裡。
4. **除錯（constitution VI）**：修 bug 一律先寫出重現問題的失敗測試，針對根因修正，根因寫進 commit 訊息。原因還不清楚時——測試以非預期的方式失敗、既有測試轉紅、問題時有時無、或第一次修正後紅燈沒有轉綠——先載入 `diagnosing-bugs` skill，照它的步驟做出穩定重現的紅燈再找原因。
5. **實作中發現需求問題 → 停手，退回 Phase 2**，改 spec、過 grill，再繼續。subagent 發現時停手並回報，由主 agent 退回。需求變更一律經過 spec——spec 和實作不同步是整個流程最大的失效模式。
6. **開 PR 前跑 `/speckit-converge`**（完整通道）：它對照 spec、plan、tasks 檢查目前的程式碼，列出缺漏、只做一半、與 spec 牴觸、以及 spec 沒要求卻多做的部分。它不改程式碼，唯一的寫入是把剩餘工作附加到 tasks.md。
   - 回報 `Converged` → 開 PR，把這個結果寫進 PR 描述。
   - 附加了任務 → 照本階段的紀律完成後重跑，直到 `Converged`。附加的任務若其實是需求問題，照第 5 點退回 Phase 2。
   - 這是實作者的自我檢查，**不取代 Phase 5 的獨立驗收**，也不計入打回次數。
   - 輕量通道可選用：該 feature 目錄已有 plan.md 與 tasks.md 時才能執行，並以參數把範圍限定在本次增修的標準。快速通道不動 spec，不適用。

快速通道沒有 tasks.md：交辦內容就是該次任務本身（bug fix 先寫重現問題的失敗測試），第 3 點不適用。

---

## Phase 5：驗收（spec 對照）

**負責人**：PR reviewer——**不得是實作者本人**。專案無真人可用時：由 fresh-context 的獨立 agent 代位，**可自由讀取 repo**（架構偏離、效能退化、與既有模組的互動都需要 codebase 脈絡），僅不得共享實作過程的對話記錄。

1. 開 PR，reviewer 將該 spec 的驗收標準貼入 PR 描述（或指示 agent 從 spec 生成），逐條打勾：

```markdown
## Spec 驗收（對照 specs/xxx/spec.md）
- [ ] 驗收標準 1：<自動測試連結或人工驗證方式>
- [ ] 驗收標準 2：...
- [ ] 所有 edge cases 有對應測試
- [ ] 測試的 commit 早於對應的實作 commit（豁免者見 PR 的豁免段）
- [ ] 未實作 spec 範圍外的功能（scope creep 檢查）
- [ ] `/speckit-converge` 結果為 Converged（完整通道，結果附在 PR 描述）
- [ ] spec「未解事項」各項已定案——標「已接受取捨（凍結）」或轉 issue 移出 spec
- [ ] 代位 grill（AI 執行時）：本分支有 `grill:` 開頭的 commit，且每個都含該輪問題與作者答案原文；有 `grill: [補問]` commit 時只檢查補問的 commit
```

2. 每條驗收標準優先附上自動測試作為證據；無法自動化的，寫明人工驗證步驟與結果。確認測試先行（constitution I）：測試的 commit 早於對應的實作 commit，抽查時簽出該測試 commit，測試應為紅燈。抽查測試品質：revert 對應實作後測試必須轉紅，空測試視同無測試。
3. 任一條打不了勾 → 打回 Phase 4；若原因是 spec 本身有問題 → 退回 Phase 2（計數見「退回與熔斷」）。代位 grill 問答紀錄那一勾例外，處理方式見下方第 5 點。
4. 快速通道 PR 加驗：實際 diff 超過 3 檔時，判斷是否應升級通道。完整通道的 converge 那一勾：PR 描述沒有 `Converged` 的結果，或 reviewer 自己重跑 `/speckit-converge` 後附加了任務 → 打回 Phase 4，照常計入打回次數。
5. **代位 grill 問答紀錄**：AI 代位執行的 grill（輕量通道一律適用），用 `git log origin/main..HEAD --grep '^grill:'` 列出本分支的 grill commit：至少要有一個，且每個 message 都含該輪問題與作者答案原文。不符 → 由新的 fresh-context subagent 對該 spec 重跑一輪 grill 四維度，commit message 以 `grill: [補問]` 開頭並含問答原文。之後只檢查補問的 commit（`git log origin/main..HEAD --grep '^grill: \[補問\]'`）：缺問答原文的舊 commit 無法改寫，不再計入。補問只補紀錄，不計入退回／打回次數與熔斷；補問導致 spec 修改則依原規則計為 P5→P2。本項只能確認紀錄存在且格式正確，無法證明沒有漏掉的輪次。真人主持的 grill 不適用本項。
6. **未解事項定案**：合併前，spec「未解事項」段每一項必須定案——標記「已接受的取捨（凍結）」留在 spec，或轉成 issue 移出 spec。不得無狀態滾入下一次修改：未定案的項目會累積成墓地，第十次檢視三十項時 checkbox 就變蓋章了。
7. 全數通過 → 合併。Spec 文件與程式碼同 repo 留存，作為未來變更的基準。

---

## 退回與熔斷

退回路徑：P3→P2（plan 發現 spec 有洞）、P4→P2（實作發現需求問題）、P5→P4（驗收不過）、P5→P2（根因在 spec）。

退回不是免費的——反覆繞圈是「需求本身沒想清楚」的訊號，流程必須主動熔斷，而不是讓人在迴圈裡耗。退回 Phase 2 的次數（三條路徑合計）與 Phase 5→4 的打回次數，記在 PR 描述的「退回與打回」欄；開 PR 前發生的退回，開 PR 時補記。

- **同一 feature 累計退回 Phase 2 達 2 次** → 熔斷：停止流程，**強制拆分**——已想清楚的部分保留成獨立 spec 照常前進，只把糾纏不清的部分回 Phase 1 重新定義。與升級規則同一哲學：成本是重新組織，不是作廢；探索性需求退回兩次很正常，熔斷懲罰的是糾纏，不是探索。有真人：拆分決策拉高為專案級討論；無真人：換 fresh agent 從**問題定義**（而非現有 spec）主導拆分。
- **Phase 5→4 打回達 2 次** → 第三次實作前，強制先做根因分析（constitution VI）並寫入 PR，才准再動手。

---

## 失效模式與對策

| 症狀 | 根因 | 對策 |
|---|---|---|
| 把解決方案當目標，做完才發現不是 PM 要的 | 跳過目標確認直接分流 | 提案一律先過 `steelman-requirement`；目標未獲 PM 確認不寫進 spec；「bug」但 spec 未描述該期望行為 → 視為提案照跑 |
| AI 做出來的不是想要的 | Phase 2 grill 不夠深 | 檢查 grill 執行者是否符合通道規則（完整通道有真人應真人主持；AI 代位須為 fresh-context）——流於形式的檢核清單多半是自己審自己 |
| AI 回報完成但驗收失敗 | Phase 4 測試沒覆蓋驗收標準 | 驗收標準在 Phase 3 就要對應到測試任務（`/speckit-analyze` 檢查）；開 PR 前跑 `/speckit-converge` |
| Spec 和程式碼對不上 | 實作中偷改需求 | 嚴格執行「改需求必退 Phase 2」 |
| 同一 feature 反覆退回繞圈 | 需求根本沒想清楚 | 觸發熔斷：強制拆分，糾纏部分回 Phase 1 重新定義（見「退回與熔斷」） |
| 測試存在但形同虛設 | TDD 被當表格應付、寫空測試交差 | Phase 5 抽查「revert 實作測試須轉紅」；適用豁免類別就走豁免程序，不寫假測試 |
| 不同成員產出品質落差大 | 有人跳過 grill | PR review 檢查 spec 的 grill 檢核與 commit 歷史 |

---

## 附錄：工具邊界速查

| 動作 | 用什麼 | 不用什麼 |
|---|---|---|
| 目標確認（分流前） | `steelman-requirement`（對提案；bug 回報免） | 直接對提案開評（先 steelman 後評論，順序不可顛倒） |
| 產 spec / plan / tasks | `/speckit-*` | 其他 plugin 的規劃類 skill（例如 Superpowers 的 `brainstorming`、`writing-plans`） |
| 規劃完成檢查、開 PR 前自我檢查 | `/speckit-analyze`、`/speckit-converge` | 拿它們取代 Phase 5 的獨立驗收 |
| 錘鍊需求 | `grill-me`（對 spec 文件） | grill-with-docs（會另產文件） |
| 實作 | 主 agent 派 fresh-context subagent，依 Phase 4 與 constitution I | 其他 plugin 的執行類 skill（例如 Superpowers 的 `subagent-driven-development`、`executing-plans`：它們有自己的進度檔與派工規則，不勾 tasks.md） |
| 除錯 | 失敗測試＋根因修正；原因不清楚時用 `diagnosing-bugs` | 沒有重現就直接改 |
| 驗收 | PR spec 對照清單 | — |
