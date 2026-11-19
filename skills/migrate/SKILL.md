---
name: migrate
description: 把既有專案遷移到 spec-kit 1.x 與本 plugin 現行版本（spec-kit 0.x，或跑過 0.6.0 之前 setup 的專案）。
disable-model-invocation: true
---

把既有專案遷移到 spec-kit 1.x 與本 plugin 現行版本。全程在專用分支上進行，每一步先偵測、已達目標狀態就記「跳過」，所以可以重跑。會改動檔案的動作一律先徵詢。

版本行為以 0.8.10 的專案搭配 1.4 的 CLI 實測（2026-11-19），其餘依 spec-kit 的 CHANGELOG 與升級文件。

## 1. 盤點

先列出現況表，再動任何檔案：

| 項目 | 怎麼查 | 目標狀態 |
|---|---|---|
| CLI 版本 | `specify version` | ≥1.0 |
| 專案版本 | `.specify/integration.json` 的 `version`；沒有這個檔記「缺 integration 資料」 | ≥1.0 |
| 舊版模型設定 | [legacy-model-settings.md](legacy-model-settings.md) 的偵測對象 | 無舊版值 |
| extension | `.specify/extensions.yml` 的 `installed` | 已安裝者為最新 |
| `feature.json` 是否進版控 | `git ls-files --error-unmatch .specify/feature.json` | 未追蹤 |
| plugin 產物 | CLAUDE.md 的「交辦流程鐵律」段落、`.github/pull_request_template.md`、`specs/retro-log.md` 檔頭註解，各自與本 plugin `templates/` 比對 | 與 template 一致 |
| 進行中的 feature | `git branch -a --no-merged <預設分支>` 中，相對預設分支有 `specs/` 變更的分支 | 每個都有接續步驟（步驟 7） |
| 工作樹 | `git status --short` | 乾淨 |

**完成條件**：每一列都有查到的值。全部已達目標狀態 → 回報「不需遷移」並結束。

## 2. 準備

1. 工作樹有未 commit 的修改 → 請使用者先 commit 或 stash。
2. 徵詢後，從預設分支建立遷移分支（預設名稱 `chore/speckit-1x-migration`）。
3. 徵詢一次：之後每個步驟完成時各 commit 一次。分開的 commit 讓升級的 diff 看得懂，也是下一步的備份。使用者婉拒 → 不 commit，照常進行。
4. CLI 低於 1.0 → 請使用者用原本的安裝工具升級，再確認 `specify version`：`uv tool install --force specify-cli` 或 `pipx install --force specify-cli`（需 Python 3.11+；0.x 的 `specify self upgrade` 還不能用）。

**完成條件**：在遷移分支上、工作樹乾淨、`specify version` ≥1.0。CLI 升不上去 → 停在這裡，總結表列「待補」。

## 3. 清理舊版模型設定

盤點有舊版值時，照 [legacy-model-settings.md](legacy-model-settings.md) 處理。

**完成條件**：每個偵測對象的結果是「移除」「保留（團隊設定）」「保留（使用者選擇）」或「跳過」之一。

## 4. 升級 spec-kit 專案檔

1. 執行 `specify integration upgrade claude`。它保留 constitution 與 `specs/`，新增 `speckit-converge`，並移除上游已停用的 `speckit-taskstoissues`（向使用者說明）。
2. 依結果分支：
   - **成功** → 下一點。
   - **`file(s) have been modified since installation`** → 列出被擋的檔案。原因是步驟 3 保留的鍵 → 這一步列「待補」並說明：要升級就得先移除那些鍵，由使用者決定。其他本地修改 → 給使用者看 diff，由他選擇保留（列「待補」）或以 `--force` 覆蓋。
   - **`Integration 'claude' is not installed`**（缺 integration 資料）→ 徵詢後改用 `specify init --here --force --integration claude`。它保留既有的 constitution，覆寫 speckit skill 檔與共用腳本。
3. 執行 `specify integration status`。CLI 升級時會提示可加 `--force` 刷新共用檔案；status 為 `OK` 代表共用檔案已是最新，照 status 判斷。

**完成條件**：`.specify/integration.json` 的 `version` ≥1.0 且 status 為 `OK`；或這一步已列「待補」並附原因。

## 5. extension 與版控狀態

1. 徵詢後執行 `echo y | specify extension update`（這個指令要確認，沒有免確認的旗標）。
2. 在 repo 層 `.gitignore` 加上 `.specify/extensions/.cache/`（已有等效規則則跳過）。這是 extension 更新留下的目錄快取，spec-kit 自己的 `.specify/.gitignore` 沒有忽略它。
3. `.specify/feature.json` 已進版控 → 徵詢後 `git rm --cached .specify/feature.json`。1.x 把它當成每個人本機的狀態，升級產生的 `.specify/.gitignore` 已忽略它，但忽略規則對已追蹤的檔案無效。
4. `installed` 沒有 `git` → 徵詢是否 `specify extension add git`。1.x 的 `/speckit-specify` 靠這個 extension 才會自動建立 feature branch；婉拒則提醒 Phase 1 之前要自己建立。

**完成條件**：以上四點各有結果（完成、跳過或婉拒）。

## 6. 同步 plugin 產物

讀取本 plugin 根目錄的 `commands/setup.md`（本 skill 目錄往上兩層），執行其中的步驟 6 到 9：`.gitignore` 的 settings 分層、CLAUDE.md 段落、PR template、retro log。

遷移時 retro log 多做一件事：`specs/retro-log.md` 已存在時，把檔頭的說明註解換成 `templates/retro-log.md` 的版本，資料列原樣保留。

讀不到 `commands/setup.md` → 這一步列「待補」，請使用者在遷移後執行 `/spec-grill-flow:setup`。

**完成條件**：CLAUDE.md 段落、PR template、retro log 檔頭註解都與 template 一致，或已列「待補」。

## 7. 進行中的 feature

1.x 的 speckit 指令只看 `.specify/feature.json`（或環境變數 `SPECIFY_FEATURE_DIRECTORY`）決定作用在哪個 feature 目錄；0.x 會從分支名稱推斷，1.x 不會。檔案不存在時指令報錯，內容過期時指令會默默作用在另一個 feature 上。

對盤點列出的每個進行中分支，在總結裡給出接續步驟。留在遷移分支上，由使用者自己切換：

1. 遷移分支合併後，把預設分支合併進該 feature 分支。該分支改過 `.specify/feature.json` 時，合併會在這個檔案出現 modify/delete 衝突（遷移把它改成不追蹤）：執行 `git rm --cached .specify/feature.json` 後完成合併，本機的檔案會留著。
2. 寫入 `.specify/feature.json`，內容為 `{"feature_directory":"specs/<該 feature 的目錄>"}`。
3. 執行 `bash .specify/scripts/bash/check-prerequisites.sh --json --paths-only`，確認輸出的 `FEATURE_DIR` 是該 feature 的目錄。
4. 從原本所在的 Phase 繼續。完整通道且尚未開 PR 的 feature，適用新增的 `/speckit-analyze`（Phase 3）與 `/speckit-converge`（Phase 4 結尾）。

**完成條件**：每個進行中的分支都有對應的 feature 目錄與上述四步；沒有進行中的分支則記「無」。

## 8. 驗證與總結

驗證：

- `specify integration status` 為 `OK`。
- `.claude/skills/speckit-converge/SKILL.md` 存在。
- 步驟 3 的偵測對象重新檢查一次，結果與該步驟記錄的一致。

輸出總結表，每個步驟一列，結果用：完成／跳過／待補／保留（團隊設定）／保留（使用者選擇）。接著列出：

- **待補事項**與各自的原因。
- **進行中 feature 的接續步驟**（步驟 7）。
- **給團隊的說明**：
  - 每位成員合併遷移後，各自把 `specify` CLI 升到 1.x。
  - speckit 指令靠 `.specify/feature.json` 找 feature，切換分支後要確認它。
  - 完整通道新增 Phase 3 的 `/speckit-analyze` 與 Phase 4 結尾的 `/speckit-converge`。
  - 移除了舊版模型設定時：speckit 指令改為跟隨各自選擇的模型，個人偏好寫在 `.claude/settings.local.json`。
- 提醒使用者開 PR 合併遷移分支。
