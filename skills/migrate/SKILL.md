---
name: migrate
description: 把既有專案遷移到 spec-kit 1.x 與本 plugin 現行版本（spec-kit 0.x，或跑過 0.7.0 之前 setup 的專案）。
disable-model-invocation: true
---

把既有專案遷移到 spec-kit 1.x 與本 plugin 現行版本。全程在專用分支上進行，每一步先偵測、已達目標狀態就記「跳過」，所以可以重跑。會刪除檔案、覆寫使用者改過的內容、或修改 constitution 的動作，逐項先徵詢；其餘改動留在遷移分支上，由使用者從 diff 檢視。

版本行為以 0.8.10 的專案搭配 1.4 的 CLI 實測（2026-11-19），其餘依 spec-kit 的 CHANGELOG 與升級文件。

## 1. 盤點

先列出現況表，再動任何檔案：

| 項目 | 怎麼查 | 目標狀態 |
|---|---|---|
| CLI 版本 | `specify version` | ≥1.0 |
| 專案版本 | `.specify/integration.json` 的 `version`；沒有這個檔記「缺 integration 資料」 | ≥1.0 |
| 舊版模型設定 | [legacy-model-settings.md](legacy-model-settings.md) 的偵測對象 | 無舊版值 |
| extension | `.specify/extensions.yml` 的 `installed` | 沒有 `git`；其餘已安裝者為最新。選擇保留 `git` 的專案盤點時仍算未達目標，每次重跑都會再問 |
| `feature.json` 是否進版控 | `git ls-files --error-unmatch .specify/feature.json` | 未追蹤 |
| plugin 產物 | CLAUDE.md 的「交辦流程鐵律」段落、PR／MR template（位置依平台，見 setup 步驟 8；GitLab 專案沒有 `.gitlab/merge_request_templates/Default.md` 即未達目標），各自與本 plugin `templates/` 比對。比對只看文字內容，不計空行、行尾空白與換行位置的差異（專案的 formatter 可能重排過） | 與 template 一致 |
| constitution 的對照 | 專案的 constitution 與本 plugin `templates/constitution.md` 的 Core Principles I 到 VI 逐條比對，並找出與流程牴觸的條文（做法見 setup 的「既有 constitution 的對照」） | 六條都有等效的條文，且沒有牴觸的條文。選擇保留的項目盤點時仍算未達目標，每次重跑都會再問 |
| 已停用的產物 | `specs/retro-log.md` 是否存在；constitution 是否還有 [legacy-constitution.md](legacy-constitution.md) 列出的舊條文 | 沒有 `specs/retro-log.md`；舊條文都沒有，或已是專案自己改寫過的句子 |
| 進行中的 feature | `git branch -a --no-merged <預設分支>` 中，相對預設分支有 `specs/` 變更的分支 | 每個都有接續步驟（步驟 7） |
| `.gitignore` | `git check-ignore -q .specify/extensions/.cache/x`、`git check-ignore -q .claude/settings.local.json`；`.specify/extensions/.backup/` 存在時再查 `git check-ignore -q .specify/extensions/.backup/x` | 都被 repo 層的規則忽略 |
| formatter | 專案有沒有會改寫 Markdown 的 formatter：`.prettierrc*`、`prettier.config.*`、`package.json` 的 `prettier` 或 `lint-staged`、`.lintstagedrc*`、`.husky/`、`.pre-commit-config.yaml`、`lefthook.yml`、`git rev-parse --git-path hooks/pre-commit` 指到的檔、`git config core.hooksPath` 指到的目錄。有 prettier 時對 `.claude/skills/speckit-plan/SKILL.md` 與 `.specify/templates/plan-template.md` 各執行一次 `npx prettier --file-info <檔>`（一次只能給一個檔）；其他 formatter 照它自己的忽略設定判斷這兩個檔有沒有被排除 | 沒有這類 formatter，或兩個檔都被排除（prettier：輸出的 `ignored` 都是 `true`）。婉拒加入忽略清單的專案盤點時仍算未達目標，每次重跑都會再問 |
| 工作樹 | `git status --short` | 乾淨 |

**完成條件**：每一列都有查到的值。全部已達目標狀態 → 回報「不需遷移」並結束。

## 2. 準備

1. 工作樹有未 commit 的修改 → 請使用者先 commit 或 stash。
2. 徵詢後，從預設分支建立遷移分支（預設名稱 `chore/spec-grill-flow-migration`）。分支已存在（上次沒做完）→ 切過去接著做。
3. 徵詢一次：之後每個步驟完成時各 commit 一次。commit 訊息照專案的 commit 慣例寫；專案有 commitlint 時要過得了（例如 `chore: remove spec-kit git extension`）。分開的 commit 讓升級的 diff 看得懂，也是下一步的備份。使用者婉拒 → 不 commit，照常進行。
4. CLI 低於 1.0 → CLI 是使用者機器上的全域工具，升級由使用者自己執行：給出指令後停下來等他回報，再以 `specify version` 確認。指令是 `uv tool install --force specify-cli` 或 `pipx install --force specify-cli`，用他原本的安裝工具（需 Python 3.11+；0.x 的 `specify self upgrade` 還不能用）。使用者事先說過「一律同意」也一樣在這裡停下。

**完成條件**：在遷移分支上、工作樹乾淨、`specify version` ≥1.0。CLI 升不上去 → 停在這裡，總結表列「待補」。

## 3. 清理舊版模型設定

盤點有舊版值時，照 [legacy-model-settings.md](legacy-model-settings.md) 處理。

**完成條件**：每個偵測對象的結果是「移除」「保留（團隊設定）」「保留（使用者選擇）」或「跳過」之一。

## 4. 升級 spec-kit 專案檔

1. **先讓 formatter 放過 spec-kit 的檔案**：盤點的 formatter 一列未達目標 → 徵詢後把 `.specify/` 與 `.claude/skills/speckit-*/` 加進它的忽略清單（prettier 是 `.prettierignore`）。spec-kit 以檔案雜湊判斷受管檔案有沒有被改過，formatter 多半由 lint-staged 在 commit 時執行。檔案被重排之後 `specify integration status` 停在 WARNING；這次與之後的每次升級，`.claude/skills/speckit-*` 的檔案會擋下整個升級，`.specify/` 下的共用檔則被跳過、留著舊內容（2026-11-20 以 0.8.10 升 1.4.3 實測）。婉拒 → 列「保留（使用者選擇）」，並說明代價：commit 時 formatter 會把剛升級的檔案再排一次，status 回到 WARNING，下次升級遇到同樣的情況。沒有這類 formatter → 跳過。
2. **升級前列出使用者改過的 spec-kit 檔**。升級指令會重新產生 `.claude/skills/speckit-*`、`.specify/templates/`、`.specify/scripts/` 下的檔案，不會逐檔詢問，所以要在執行前列齊。來源有三個，取聯集：
   - **雜湊**：`specify integration status --json` 的 `manifests.claude.modified_files`（核心 skill 檔）與 `manifests.speckit.modified_files`（`.specify/` 下的共用檔）。只看升級的訊息不夠：skill 檔被改過時升級會中止，訊息只列 skill 檔；共用檔被改過時升級不擋，只回報 `Preserved` 並留著舊內容；而 `--force` 會把兩類一起覆蓋，沒列在訊息裡的也一樣。
   - **git 歷史**，給 spec-kit 沒有記錄雜湊的檔用。這種檔有兩類：extension 的 skill 檔（`.claude/skills/speckit-<extension id>-*`，id 取自 `.specify/extensions.yml` 的 `installed`；任何升級與 `specify extension update` 都會重新產生它們，不論有沒有 `--force`）；以及缺 integration 資料的專案的全部檔案（盤點記「缺 integration 資料」；`status --json` 的 `manifests` 是空的，Findings 有 `integration-state-missing`）。以最近一次安裝或升級的 commit 為基準：`B=$(git log -1 --format=%H -- .specify/init-options.json)`，再以 `git diff --diff-filter=M --name-only $B HEAD -- '.claude/skills/speckit-*' .specify/templates .specify/scripts` 列出之後被改過的檔。這次遷移自己的 commit（步驟 3、忽略清單）造成的差異不算；雜湊已涵蓋的檔以雜湊為準。
   - **步驟 3 保留了鍵的檔**一律列入。

   git 歷史看不到與安裝同一個 commit 的修改，沒有 `.specify/init-options.json` 時也找不到基準。專案有 extension 或缺 integration 資料時，徵詢時明講這個限制，並請使用者指出清單以外他們改過的檔。

   清單是空的，使用者也沒有補充 → 跳到第 4 點。
3. **逐檔決定**，每個檔都要有決定。能選什麼依檔案的種類，徵詢前先說明代價：
   - **核心 skill 檔**（缺 integration 資料時：不屬於任何 extension 的 `speckit-*`）→ 覆蓋或保留。保留任何一個，spec-kit 就不能升級，這一步與步驟 5、6 都列「待補」。
   - **共用檔** → 覆蓋或保留。保留的檔停在現在的內容，拿不到新版對它的更新。
   - **extension 的 skill 檔** → 覆蓋，或升級後把使用者的修改重新套到新版上。不提供整檔保留：新版 extension 可能改了腳本的檔名，舊版 skill 檔會指向不存在的腳本（git extension 1.0.0 升 1.0.8 就是這樣，2026-11-23 實測）。

   各檔的判斷：
   - 步驟 3 保留的鍵 → 保留。
   - 只有格式差異（formatter 重排過）→ 覆蓋，告知使用者即可，不必逐檔看 diff。兩件事都成立才算：
     1. **找得到原始版本**：該檔的 git 歷史裡，有一個版本的 SHA-256 等於 manifest 記錄的雜湊（`.specify/integrations/<claude 或 speckit>.manifest.json` 的 `files`）。對 `git log --format=%H -- <檔>` 列出的每個 commit 檢查 `git show <commit>:<檔> | sha256sum`。
     2. **原始版本排過之後等於現在的檔案**：用專案的 formatter 排那個版本，與工作目錄現在的檔案比對。prettier 的寫法：`git show <該 commit>:<檔> | npx prettier --ignore-path /dev/null --stdin-filepath <檔> | diff -q - <檔>`，沒有輸出即相同。`--ignore-path /dev/null` 不能省：第 1 點剛把這些檔加進忽略清單，少了它 prettier 會原樣輸出，比對結果一律是不同。
   - 其他本地修改，或找不到原始版本 → 給使用者看 diff，由他選擇覆蓋或保留。找不到原始版本是常見結果：formatter 在 commit 時執行的專案，安裝那次 commit 進歷史的就已經是排過的內容，無從證明只有格式差異；這時沒有 diff 可看，改列出該檔的歷史（`git log --oneline -- <檔>`）。
   - 沒有雜湊的檔 → 給使用者看 `git diff $B HEAD -- <檔>`，由他選擇。
4. **升級**。升級會保留 constitution 與 `specs/`，新增 `speckit-converge`。依第 3 點的決定選做法，由上往下取第一個符合的：
   - 有核心 skill 檔要保留 → 不升級，這一步列「待補」並說明：要升級就得先放棄那些修改（步驟 3 保留的鍵就是移除它們）。使用者改變主意 → 回到第 3 點重新決定。
   - 缺 integration 資料 → 徵詢後執行 `specify init --here --force --integration claude`（這種專案執行 `specify integration upgrade claude` 只會回報 `Integration 'claude' is not installed`）。它保留既有的 constitution，覆寫 speckit skill 檔與共用檔。婉拒 → 這一步列「待補」。
   - `modified_files` 兩個清單都是空的 → `specify integration upgrade claude`。
   - 其餘 → `specify integration upgrade claude --force`。

   升級之後：
   - 要保留的共用檔 → 以 `git restore --source=HEAD -- <檔>` 還原，這一步列「保留（使用者選擇）」（status 會是 WARNING）。
   - 要重新套用修改的 extension skill 檔 → 把第 2 點那份 diff 的修改搬到新產生的檔上。搬不上去，或新版已經改掉那一段 → 給使用者看兩邊，由他決定。搬完確認 skill 檔裡引用的腳本路徑都存在。這些檔不影響 status，這一步照常列完成，在總結註明。重新套用的修改另外 commit，不和升級放在同一個 commit：下次升級要靠 git 歷史找到它（步驟 2 選擇不 commit 時，在總結提醒使用者分開 commit）。
   - 上游已停用的 `speckit-taskstoissues`：`upgrade` 會移除它（向使用者說明）；`init` 不會，徵詢後刪除 `.claude/skills/speckit-taskstoissues/`。
5. 執行 `specify integration status`。CLI 升級時會提示可加 `--force` 刷新共用檔案；status 為 `OK` 代表共用檔案已是最新，照 status 判斷。
6. **這一步 commit 之後再查一次** `specify integration status --json`：被改過的檔比 commit 之前多，代表有盤點沒偵測到的 formatter 在 commit 時重排了檔案 → 徵詢後把 spec-kit 的檔案加進它的忽略清單，再執行一次 `specify integration upgrade claude --force`（要保留的共用檔同樣還原；重新套用過的修改已在 HEAD，以 `git restore --source=HEAD -- <檔>` 取回）、commit，確認沒有再多出被改過的檔。婉拒 → 比照第 1 點列「保留（使用者選擇）」。兩種情況不做這項檢查：第 1 點婉拒忽略清單（多出來的檔是預期結果）；步驟 2 選擇不 commit（在總結提醒使用者 commit 之後自己執行一次 `specify integration status`）。

**完成條件**：`.specify/integration.json` 的 `version` ≥1.0 且 status 為 `OK`；或這一步已列「待補」並附原因。使用者選擇保留時——第 1 點或第 6 點婉拒忽略清單（commit 之後 formatter 會重排剛升級的檔案），或第 4 點保留了共用檔的本地修改——這一步列「保留（使用者選擇）」；status 是 `OK`，或是 `WARNING` 且 Findings 只有 `managed-files-modified`，都算完成。「待補」與「保留（使用者選擇）」同時成立時列「待補」。列「待補」時，步驟 5 與 6 也列「待補」並直接到步驟 7：extension 更新與 plugin 產物都以 1.x 的專案檔為前提，先做會讓專案停在新舊混用的狀態。

## 5. extension 與版控狀態

1. **git extension**：`installed` 有 `git` → 說明下面的理由並徵詢，建議移除。沒有 → 跳過。
   - 理由：本流程在 Phase 1 依專案慣例建立分支。這個 extension 在 `/speckit-specify` 之前有一個必跑的 hook，會先開一個 `<編號>-<名稱>` 的分支；它的格式只能加前綴，最後一段一定以編號開頭，配合不了其他命名慣例（例如工單編號）。
   - 移除的代價：feature 編號之後只看本地的 `specs/` 目錄，不再參考遠端分支；兩個人同時開新 feature 可能拿到同一個編號（目錄名與分支名因後綴不同不會衝突）。徵詢時一併說明。
   - 移除 → `echo y | specify extension remove git`。hook 與 `speckit-git-*` skill 會一併移除（步驟 4 重新套用到 `speckit-git-*` 的修改也跟著消失，徵詢時說明），設定檔備份到 `.specify/extensions/.backup/git/`。用 remove 而不是 disable：disable 只停用 hook，skill 檔還留在 `.claude/skills/`。
   - 保留 → 只在專案的分支慣例就是 `<編號>-<名稱>` 時合理。這時 hook 會從當下的分支開出新分支，Phase 1 要先切到基準分支再執行 `/speckit-specify`（見 team-workflow 的 Phase 1）。列「保留（使用者選擇）」；盤點仍把它算未達目標，重跑時會再問。
2. 還有其他已安裝的 extension（或上一點選擇保留）→ 徵詢後執行 `echo y | specify extension update`（這個指令要確認，沒有免確認的旗標）。它會重新產生 extension 的 skill 檔；步驟 4 重新套用過修改的檔，更新後照同樣方式再套一次、確認引用的腳本路徑都存在，同樣另外 commit。沒有 → 跳過。
3. 在 repo 層 `.gitignore` 加上 `.specify/extensions/.cache/`（已有等效規則則跳過）。這是 extension 更新留下的目錄快取，spec-kit 自己的 `.specify/.gitignore` 沒有忽略它。`.specify/extensions/.backup/` 存在時（移除 extension 留下的設定備份）也一併加上。
4. `.specify/feature.json` 已進版控 → 徵詢後 `git rm --cached .specify/feature.json`。1.x 把它當成每個人本機的狀態，升級產生的 `.specify/.gitignore` 已忽略它，但忽略規則對已追蹤的檔案無效。

**完成條件**：以上四點各有結果（完成、跳過、婉拒或保留（使用者選擇））；選擇移除時，`.specify/extensions.yml` 裡不再有 `speckit.git`。

## 6. 同步 plugin 產物

讀取本 plugin 根目錄的 `commands/setup.md`（本 skill 目錄往上兩層），執行其中的步驟 6 到 8：`.gitignore` 的 settings 分層、CLAUDE.md 段落、PR template。步驟 7 寫 CLAUDE.md 段落之前，先照同一個檔案的「既有 constitution 的對照」處理盤點時不是「有」的原則與牴觸的條文。

接著處理 0.7.0 起停用的產物，每一項都先說明再徵詢：

1. **`specs/retro-log.md`**：流程已取消合併後的 retro，不再讀寫這個檔。徵詢使用者：刪除（`git rm`），或封存作為歷史紀錄（`git mv specs/retro-log.md specs/retro-log-archive.md`）。封存後 setup 與 migrate 不再偵測到它，重跑不會再問。兩者都婉拒 → 原樣留著，列「保留（使用者選擇）」，重跑時會再問。
2. **constitution 的舊條文**：照 [legacy-constitution.md](legacy-constitution.md) 逐項比對並徵詢。constitution 是專案自己的文件，使用者婉拒就原樣保留。

讀不到 `commands/setup.md` → 這三項與 constitution 的對照列「待補」，請使用者在遷移後執行 `/spec-grill-flow:setup`；下面兩項停用產物照常處理，它們不靠 setup。

**完成條件**：`.gitignore` 的 settings 分層、CLAUDE.md 段落、PR template 各自完成或已列「待補」；constitution 的對照六條各有結果（有、補上條文、保留（使用者選擇）），牴觸的條文各有結果（修訂條文、保留（使用者選擇）、無），或已列「待補」；retro-log 的結果是完成（刪除或封存）、保留（使用者選擇）或跳過（不存在）；constitution 舊條文三項各有 legacy-constitution.md 定義的結果。

## 7. 進行中的 feature

1.x 的 speckit 指令只看 `.specify/feature.json`（或環境變數 `SPECIFY_FEATURE_DIRECTORY`）決定作用在哪個 feature 目錄；0.x 會從分支名稱推斷，1.x 不會。檔案不存在時指令報錯，內容過期時指令會默默作用在另一個 feature 上。

對盤點列出的每個進行中分支，在總結裡給出接續步驟。留在遷移分支上，由使用者自己切換：

1. 遷移分支合併後，把預設分支合併進該 feature 分支。遷移把 `.specify/feature.json` 改成不追蹤，所以這個檔在合併時有兩種結果：
   - 該分支改過它 → 出現 modify/delete 衝突。執行 `git rm --cached .specify/feature.json` 後完成合併，本機的檔案會留著。
   - 該分支沒改過它 → 合併不會有任何提示，直接把它從工作目錄刪掉。
2. 兩種結果都重新寫入 `.specify/feature.json`，內容為 `{"feature_directory":"specs/<該 feature 的目錄>"}`。
3. 執行 `bash .specify/scripts/bash/check-prerequisites.sh --json --paths-only`，確認輸出的 `FEATURE_DIR` 是該 feature 的目錄。
4. 從原本所在的 Phase 繼續。完整通道且尚未開 PR 的 feature，適用新增的 `/speckit-analyze`（Phase 3）與 `/speckit-converge`（Phase 4 結尾）。

**完成條件**：每個進行中的分支都有對應的 feature 目錄與上述四步；沒有進行中的分支則記「無」。

## 8. 驗證與總結

驗證（步驟 4 列「待補」時，前兩項也記「待補」，不算失敗）：

- `specify integration status` 為 `OK`。步驟 4 列「保留（使用者選擇）」時，`WARNING` 且 Findings 只有 `managed-files-modified` 也算通過。
- `.claude/skills/speckit-converge/SKILL.md` 存在。
- 步驟 3 的偵測對象重新檢查一次，結果與該步驟記錄的一致。

輸出總結表，每個步驟一列，結果用：完成／跳過／待補／保留（團隊設定）／保留（使用者選擇）。接著列出：

- **待補事項**與各自的原因。
- **進行中 feature 的接續步驟**（步驟 7）。
- **給團隊的說明**：
  - 每位成員合併遷移後，各自把 `specify` CLI 升到 1.x。
  - speckit 指令靠 `.specify/feature.json` 找 feature：每次開始一項工作就重設它（見 team-workflow 的分流），新建的 worktree 要先寫入它。把預設分支合併進進行中的分支時，這個檔可能被默默刪掉，合併後重新寫入。
  - feature branch 由流程在 Phase 1 建立，命名照專案慣例；不需要 spec-kit 的 git extension。移除它之後，feature 編號只看本地的 `specs/` 目錄，同時開新 feature 的人可能拿到同一個編號。
  - 完整通道新增 Phase 3 的 `/speckit-analyze` 與 Phase 4 結尾的 `/speckit-converge`。
  - Phase 4 改由主 agent 派 fresh-context subagent 實作，一個 agent 一個 worktree：新行為的測試 commit 排在實作之前，事後新增或修改的斷言附失敗證據；任務在完成它的 PR 裡勾選；原因不清楚的 bug 用 `diagnosing-bugs`。
  - 流程規定的 commit 訊息改成過得了 commitlint 預設規則的寫法：代位 grill 的紀錄以 `docs: grill 第 N 輪` 開頭、補問以 `docs: grill [補問]` 開頭（原本是 `grill:`；先前寫下的舊紀錄仍然算數），這兩種開頭不給其他 commit 用，內文每行不超過 100 字元；任務編號放主旨結尾的括號裡。
  - 開 PR／MR 時以專案的範本作為描述骨架。
  - 流程不再依賴 Superpowers，也取消了合併後的 retro。個人仍安裝 Superpowers 的成員，CLAUDE.md 鐵律已寫明它的規劃與執行類 skill 不在本流程使用。
  - 移除了舊版模型設定時：speckit 指令改為跟隨各自選擇的模型，個人偏好寫在 `.claude/settings.local.json`。
- 提醒使用者開 PR 合併遷移分支。
