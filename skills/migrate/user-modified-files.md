# 使用者改過的 spec-kit 檔

下表的指令會覆蓋 spec-kit 的檔案，不會逐檔詢問。執行其中任何一個之前，先照「執行之前」列出使用者改過的檔並逐檔取得決定；執行之後照「執行之後」處理。

| 指令 | 使用者改過也會被覆蓋的檔 |
|---|---|
| `specify integration upgrade claude` | extension 的 skill 檔。核心 skill 檔被改過時整個升級中止；共用檔被改過時跳過該檔、留著舊內容（回報 `Preserved`） |
| `specify integration upgrade claude --force` | extension 的 skill 檔、核心 skill 檔、共用檔 |
| `specify init --here --force --integration claude` | 同上 |
| `specify extension update`（只在 extension 有新版時） | 有新版的 extension 的 skill 檔，以及它在 `.specify/extensions/<id>/` 下的 commands 與 scripts（設定檔保留） |

名詞：

- **核心 skill 檔**：`.claude/skills/speckit-*` 裡不屬於任何 extension 的檔。
- **共用檔**：`.specify/templates/`、`.specify/scripts/` 下的檔。
- **extension 的檔**：`.claude/skills/speckit-<extension id>-*` 與 `.specify/extensions/<extension id>/` 下的 commands、scripts；id 取自 `.specify/extensions.yml` 的 `installed`。

（以上行為 2026-11-20 至 23 以 0.8.10 的專案搭配 1.4.3 的 CLI 實測。）

## 執行之前

1. **記下現在的 commit**：`P=$(git rev-parse HEAD)`。之後的還原與比對都以 P 為來源，不用 HEAD：中途 commit 之後，HEAD 就不是執行前的內容了。
2. **列出使用者改過的檔**，只看這次要執行的指令會覆蓋的檔。來源有三個：
   - **spec-kit 記錄了雜湊的檔**：`specify integration status --json` 的 `manifests.claude.modified_files`（核心 skill 檔）與 `manifests.speckit.modified_files`（共用檔）。這份清單可靠。缺 integration 資料的專案沒有它（`manifests` 是空的，Findings 有 `integration-state-missing`）。
   - **步驟 3 保留了鍵的檔**。
   - **spec-kit 沒有記錄雜湊的檔**：extension 的檔；缺 integration 資料的專案則是上表列的全部。這些檔有沒有被改過，工具無從判斷，一律問使用者：把會被覆蓋的檔列給他，說明指令會重新產生它們，問有沒有手改過、想留下的。附上線索——`git log --oneline -- <這些檔的路徑>` 裡不像安裝、升級或整批排版的 commit，以及它們動到的檔。線索不保證齊全（與安裝同一個 commit 的修改就看不出來），所以沒有線索也要問。
3. **逐檔決定**，每個檔都要有決定。能選什麼依檔案的種類，徵詢前先說明代價：
   - **核心 skill 檔** → 覆蓋或保留。保留任何一個，spec-kit 就不能升級，步驟 4 與步驟 5、6 都列「待補」。
   - **共用檔** → 覆蓋，或保留修改。保留有兩種做法，由使用者選：執行後把修改重新套到新版上（建議；拿得到新版對這個檔的更新），或整檔還原（停在現在的內容）。
   - **extension 的檔** → 覆蓋，或執行後把修改重新套到新版上。不提供整檔還原：新版 extension 可能改了腳本的檔名，舊版 skill 檔會指向不存在的腳本（git extension 1.0.0 升 1.0.8 就是這樣）。

   各檔的判斷：
   - 步驟 3 保留的鍵 → 保留。
   - 有雜湊的檔，只有格式差異（formatter 重排過）→ 覆蓋，告知使用者即可，不必逐檔看 diff。兩件事都成立才算：
     1. **找得到原始版本**：該檔的 git 歷史裡，有一個版本的 SHA-256 等於 manifest 記錄的雜湊（`.specify/integrations/<claude 或 speckit>.manifest.json` 的 `files`）。對 `git log --format=%H -- <檔>` 列出的每個 commit 檢查 `git show <commit>:<檔> | sha256sum`。
     2. **原始版本排過之後等於現在的檔案**：用專案的 formatter 排那個版本，與工作目錄現在的檔案比對。prettier 的寫法：`git show <該 commit>:<檔> | npx prettier --ignore-path /dev/null --stdin-filepath <檔> | diff -q - <檔>`，沒有輸出即相同。`--ignore-path /dev/null` 不能省：這些檔已經在忽略清單裡，少了它 prettier 會原樣輸出，比對結果一律是不同。
   - 有雜湊的檔，其他本地修改或找不到原始版本 → 給使用者看 diff，由他選擇。找不到原始版本是常見結果：formatter 在 commit 時執行的專案，安裝那次 commit 進歷史的就已經是排過的內容，無從證明只有格式差異；這時沒有 diff 可看，改列出該檔的歷史（`git log --oneline -- <檔>`）。
   - 沒有雜湊的檔，使用者說改過 → 給他看 `git log -p -- <檔>`，確認要留下的是哪些修改，由他選擇。

## 執行之後

依序做：

1. **整檔還原**的共用檔 → `git restore --source=$P -- <檔>`。
2. 步驟 2 選擇 commit 時，先 commit 指令產生的結果。重新套用的修改放在下一個 commit，歷史裡才看得出哪些是使用者的修改。
3. **重新套用**修改的檔 → 比對 `git show $P:<檔>`（執行前、含使用者修改的版本）與新產生的檔，把上面第 3 點確認要留下的修改搬到新檔上。
   - 搬不上去，或新版已經改掉那一段 → 給使用者看兩邊，由他決定。
   - skill 檔搬完後，確認它引用的腳本路徑都存在；有不存在的 → 給使用者看，由他決定。
   - 步驟 2 選擇 commit 時，commit 這些修改。
4. **結果**：保留了共用檔的修改（兩種做法都算）→ `specify integration status` 會是 WARNING，該步驟列「保留（使用者選擇）」。只動到 extension 的檔 → 不影響 status，該步驟照常列完成。兩種都在總結註明保留了哪些檔的哪些修改。
