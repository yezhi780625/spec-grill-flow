# 使用者改過的 spec-kit 檔

下表的指令會覆蓋 spec-kit 的檔案，不會逐檔詢問。執行其中任何一個之前，先照「執行之前」列出使用者改過的檔並逐檔取得決定；執行之後照「執行之後」處理。

| 指令 | 使用者改過也會被覆蓋的檔 |
|---|---|
| `specify integration upgrade claude` | extension 的 skill 檔。核心 skill 檔被改過時整個升級中止；共用檔被改過、或已經存在而 manifest 沒有記錄時，跳過該檔、留著舊內容（回報 `Preserved`） |
| `specify integration upgrade claude --force` | extension 的 skill 檔、核心 skill 檔、共用檔 |
| `specify init --here --force --integration claude` | 同上 |
| `specify extension update`（只在 extension 有新版時） | 有新版的 extension 的 skill 檔；`.specify/extensions/<id>/` 下除了設定檔以外的全部檔；`.specify/extensions.yml` 裡這個 extension 的 hook 設定（重設成預設值，例如 `enabled: false` 變回 `true`） |

名詞：

- **核心 skill 檔**：`.claude/skills/speckit-*` 裡不屬於任何 extension 的檔。
- **共用檔**：`.specify/templates/` 下的範本、`.specify/scripts/` 下的腳本，以及 `.specify/.gitignore`。範本以外的，以下都照腳本的規則處理。
- **extension 的檔**：skill 檔（`.claude/skills/speckit-<extension id>-*`）與 `.specify/extensions/<extension id>/` 下設定檔以外的檔；id 取自 `.specify/extensions.yml` 的 `installed`，設定檔是 `<id>-config.yml`。skill 檔由 `.specify/extensions/<id>/commands/` 下的來源檔產生，`specify integration upgrade` 每次都從來源檔重新產生它們（`speckit-git-commit` 的來源檔是 `speckit.git.commit.md`）。

（以上行為 2026-11-20 至 23 以 0.8.10 的專案搭配 1.4.3 的 CLI、git extension 1.0.0 升 1.0.8 實測。）

## 執行之前

1. **記下現在的 commit**：執行 `git rev-parse HEAD`，記下輸出的 SHA，以下稱 P。之後的指令直接代入這個 SHA，不要存成 shell 變數——每次執行指令都是新的 shell，變數不會留著。還原與比對都以 P 為來源，不用 HEAD：commit 之後，HEAD 就不是執行前的內容了。
2. **列出使用者改過的檔**，只看這次要執行的指令會覆蓋的檔。來源有三個：
   - **spec-kit 記錄了雜湊的檔**：`specify integration status --json` 的 `manifests.claude.modified_files`（核心 skill 檔）與 `manifests.speckit.modified_files`（共用檔）。這份清單可靠。缺 integration 資料的專案沒有它（`manifests` 是空的，Findings 有 `integration-state-missing`）。
   - **步驟 3 保留了鍵的檔**。
   - **spec-kit 沒有記錄雜湊的檔**：extension 的檔；專案裡已經存在、但 manifest 沒有記錄的共用檔（`git ls-files .specify/templates .specify/scripts .specify/.gitignore` 列出的檔裡，不在 `speckit.manifest.json` 的 `files` 的那些。新版有同名檔才會被覆蓋，例如使用者自建的 `.specify/.gitignore`；使用者自己加的其他檔不會被動到，問的時候說「可能被覆蓋」）；缺 integration 資料的專案則是上表列的全部。這些檔有沒有被改過，工具無從判斷，一律問使用者：把會被覆蓋的檔列給他，說明指令會重新產生它們，問有沒有手改過、想留下的。附上線索——`git log --oneline -- <這些檔的路徑>` 裡不像安裝、升級或整批排版的 commit，以及它們動到的檔。線索不保證齊全（與安裝同一個 commit 的修改就看不出來），所以沒有線索也要問。

   `.specify/extensions.yml` 的 hook 設定不必問：執行之後比對得出來，見「執行之後」第 3 點。
3. **逐檔決定**，每個檔都要有決定。能選什麼依檔案的種類，徵詢前先說明代價：
   - **核心 skill 檔** → 覆蓋或保留。保留任何一個，spec-kit 就不能升級，步驟 4 與步驟 5、6 都列「待補」。
   - **範本** → 覆蓋，或保留修改。保留有兩種做法，由使用者選：執行後把修改重新套到新版上（建議；拿得到新版對這個檔的更新），或整檔還原（停在現在的內容）。
   - **腳本** → 覆蓋，或執行後把修改重新套到新版上。不提供整檔還原：新版的其他腳本會呼叫舊版腳本沒有的函式（把 0.8.10 的 `common.sh` 還原到 1.4.3 的專案，`create-new-feature.sh` 就會失敗）。
   - **extension 的檔** → 覆蓋，或執行後把修改重新套到新版上。不提供整檔還原：新版 extension 可能改了腳本的檔名，舊版 skill 檔會指向不存在的腳本（git extension 1.0.0 升 1.0.8 就是這樣）。
   - **manifest 沒記錄的既有共用檔** → 除了上面的選項，還可以選「留著」：執行後 `git diff <P> -- <檔>` 沒有輸出，代表新版沒有同名檔、它沒被動到，什麼都不用做；有輸出就照重新套用處理。
   - 腳本或 extension 的檔，上面的做法使用者都不接受 → 不執行這個指令，該步驟列「待補」。

   各檔的判斷：
   - 步驟 3 保留的鍵 → 保留。
   - 有雜湊的檔，只有格式差異（formatter 重排過）→ 覆蓋，告知使用者即可，不必逐檔看 diff。兩件事都成立才算：
     1. **找得到原始版本**：該檔的 git 歷史裡，有一個版本的 SHA-256 等於 manifest 記錄的雜湊（`.specify/integrations/<claude 或 speckit>.manifest.json` 的 `files`）。對 `git log --format=%H -- <檔>` 列出的每個 commit 檢查 `git show <commit>:<檔> | sha256sum`。
     2. **原始版本排過之後等於現在的檔案**：用專案的 formatter 排那個版本，與工作目錄現在的檔案比對。prettier 的寫法：`git show <該 commit>:<檔> | npx prettier --ignore-path /dev/null --stdin-filepath <檔> | diff -q - <檔>`，沒有輸出即相同。`--ignore-path /dev/null` 不能省：這些檔已經在忽略清單裡，少了它 prettier 會原樣輸出，比對結果一律是不同。
   - 有雜湊的檔，其他本地修改或找不到原始版本 → 給使用者看 diff，由他選擇。找不到原始版本是常見結果：formatter 在 commit 時執行的專案，安裝那次 commit 進歷史的就已經是排過的內容，無從證明只有格式差異；這時沒有 diff 可看，改列出該檔的歷史（`git log --oneline -- <檔>`）。
   - 沒有雜湊的檔，使用者說改過 → 給他看 `git log -p -- <檔>`，確認要留下的是哪些修改，由他選擇。

## 執行之後

依序做，全部做完才 commit：

1. **整檔還原**的範本 → `git restore --source=<P> -- <檔>`。
2. **重新套用**修改的檔 → 比對 `git show <P>:<檔>`（執行前、含使用者修改的版本）與新產生的檔，把「執行之前」第 3 點確認要留下的修改搬到新檔上。
   - 搬不上去，或新版已經改掉那一段 → 給使用者看兩邊，由他決定。
   - extension 的 skill 檔：同一份修改也寫進它的來源檔。只改 skill 檔的話，下一次 `specify integration upgrade` 從來源檔重新產生它，修改就消失。
   - 搬完後檢查：skill 檔引用的腳本路徑都存在；`.sh` 通過 `bash -n <檔>`，其他語言的腳本用該語言的語法檢查（例如 `python3 -m py_compile <檔>`）。沒過 → 給使用者看，由他決定。
3. 執行的是 `specify extension update` 時 → 看 `git diff <P> -- .specify/extensions.yml`。只處理一種差異：同一個 hook 在 P 是使用者改過的值（例如 `enabled: false`），現在變回預設值 → 改回使用者的值。其他差異不動：指令新增的鍵留著；同一個步驟裡移除 extension 造成的刪除不要加回去。
4. commit 由呼叫這份文件的步驟做（步驟 2 選擇 commit 時）：指令的結果與上面三點放在同一個 commit，訊息裡寫明保留了哪些檔的修改。上面三點還沒做完就不要 commit——中途停下來時工作樹是髒的，重跑時步驟 2 會認出這是沒做完的遷移，把這些檔退回執行前的內容重做。
5. **結果**：
   - 保留了共用檔的修改（不論哪種做法）→ `specify integration status` 會是 WARNING，該步驟列「保留（使用者選擇）」。
   - 只動到 extension 的檔 → 不影響 status，該步驟照常列完成。
   - 兩種都在總結寫明保留了哪些檔的哪些修改，並提醒之後自行執行指令時的後果：`specify extension update` 會再覆蓋 extension 的檔，也會把 `.specify/extensions.yml` 的 hook 設定重設成預設值；帶 `--force` 的升級與 `init` 會再覆蓋共用檔。
