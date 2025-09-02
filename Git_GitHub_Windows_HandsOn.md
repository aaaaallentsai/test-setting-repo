# Git & GitHub（Windows）從 0 開始：安裝、設定、上傳、下載、日常工作流
版本：2025-09-02

---

## 導讀與說明
本文是給 Windows 使用者的「一步一步操作手冊」。你可以從完全不懂 Git/GitHub，到完成：
1) 安裝 Git
2) 設定你的 Git 身分
3) 建立 SSH 金鑰並綁定 GitHub
4) 在 GitHub 建立 Repository（repo）
5) 在本機初始化專案、提交、推上 GitHub
6) 下載（clone）與日常 push/pull 流程
7) 新電腦快速接手你的 GitHub 專案（案例教學）

所有指令預設在「Windows PowerShell」中執行（按 Win → 輸入 PowerShell → 開啟）。

---

## 一、安裝 Git（Windows）
一、安裝 Git（Windows）
A) 先檢查是否已安裝：
  PowerShell 輸入：
    git --version
    where git
  若有版本號，表示已安裝。

B) 若未安裝，建議使用 winget（Windows 10/11 通常內建）：
    winget install --id Git.Git -e --source winget
  安裝完成後關閉並重新打開 PowerShell，再次檢查：
    git --version
    where git

（若沒有 winget，可至官網下載「Git for Windows」安裝器，沿用預設值一路 Next 即可。）

---

## 二、Git 全域設定
二、Git 全域一次性設定（必要）
設定你的身分、換行與預設分支名稱：
  git config --global user.name  "你的名字"
  git config --global user.email "你的Email（最好是 GitHub 註冊的那個）"
  git config --global core.autocrlf true
  git config --global init.defaultBranch main

檢查設定：
  git config --global --list

---

## 三、SSH 金鑰與 GitHub 綁定
三、建立 SSH 金鑰並綁定 GitHub（推薦，少詢問密碼又更安全）
1) 產生一把專門給 GitHub 用的金鑰（不覆蓋你可能已有的學校伺服器金鑰）：
  ssh-keygen -t ed25519 -C "你的GitHub Email" -f "$env:USERPROFILE\.ssh\id_ed25519_github"
  （遇到 passphrase 可直接按 Enter 空白；新手較方便）

2) 複製公鑰內容到剪貼簿：
  Get-Content "$env:USERPROFILE\.ssh\id_ed25519_github.pub" | Set-Clipboard

3) 登入 GitHub → 右上角頭像 → Settings → SSH and GPG keys → New SSH key → 貼上剛複製的內容 → 儲存

4) 啟動 ssh-agent 並加入私鑰（讓系統記住它）：
  Set-Service ssh-agent -StartupType Automatic
  Start-Service ssh-agent
  ssh-add "$env:USERPROFILE\.ssh\id_ed25519_github"

5) 新增/編輯 SSH 設定檔，確保連 github.com 使用這把鑰：
  notepad "$env:USERPROFILE\.ssh\config"
  內容（請依你的使用者名稱調整路徑，斜線可用 /）：
    Host github.com
        HostName github.com
        User git
        IdentityFile C:/Users/你的帳號/.ssh/id_ed25519_github
        IdentitiesOnly yes
        AddKeysToAgent yes

6) 測試連線：
  ssh -T git@github.com
  第一次會問要不要信任主機，輸入 yes。
  若顯示：Hi <你的帳號>! You've successfully authenticated...
  代表成功（GitHub 不提供 shell 是正常提示）。

---

## 四、在 GitHub 建立 Repository
四、在 GitHub 建立空的 Repository（repo）
1) GitHub 右上角 + → New repository
2) Repository name：輸入專案名（例如 my-first-project）
3) Visibility：Public 或 Private
4) 「不要」勾選 Add README / .gitignore / license（保持空白）
5) 建立完成後，頁面會顯示你的 SSH 位址（例如 git@github.com:你的帳號/my-first-project.git），先複製。

---

## 五、本機初始化、提交、推上 GitHub
五、在本機初始化專案、做第一次提交並推上 GitHub
# 建立資料夾並初始化
  mkdir $env:USERPROFILE\Documents\my-first-project
  cd $env:USERPROFILE\Documents\my-first-project
  git init
  git branch -M main   # 將預設分支改為 main

# 建立一個檔案示範
  Set-Content -Path README.md -Value "# My First Project"

# 納入版控並提交
  git add .
  git commit -m "first commit"

# 綁定遠端並第一次推送
  git remote add origin git@github.com:你的帳號/my-first-project.git
  git remote -v
  git push -u origin main

推送後回到 GitHub 頁面重新整理，應可看到 README.md。

---

## 六、若遠端已有 README 的處理
六、若你在 GitHub 建 repo 時有勾 README（遠端已經有內容）
第一次推會被拒，請執行：
  git pull --rebase origin main
  git push
若出現衝突，依提示修正檔案 → git add <檔案> → git rebase --continue → git push。

---

## 七、下載（clone）與日常工作流
七、下載（clone）既有專案與日常工作流
# 下載（clone）
  cd $env:USERPROFILE\Documents
  git clone git@github.com:你的帳號/目標repo.git
  cd 目標repo

# 每次開工的基本流程
  git pull --rebase           # 先把遠端最新拉回避免落後
  # 編輯檔案...
  git status                  # 看有哪些變更
  git add -A                  # 納入本次要提交的所有變更
  git commit -m "清楚描述這次修改的內容"
  git push                    # 推上 GitHub

---

## 八、HTTPS 方式（備註）
八、HTTPS 方式（備註）
若不想用 SSH，也可以用 HTTPS（remote 會長得像 https://github.com/...）。
Windows 會跳出 Git Credential Manager 視窗讓你登入並記住憑證；之後 push 不必重複登入。
但本手冊以 SSH 為主，因其更通用、跨機器時較少摩擦。

---

## 【案例】新電腦快速接手你的 repo
【案例】新電腦直接接手並上傳
情境：我換新電腦，想要把 GitHub 上的現有 repo 下載到本機、做修改、上傳。

步驟：
1) 安裝 Git 並檢查：
  git --version
  where git

2) 設定全域身分：
  git config --global user.name  "你的名字"
  git config --global user.email "你的Email"
  git config --global core.autocrlf true
  git config --global init.defaultBranch main

3) 產生並加入「這台電腦專用」的 SSH 金鑰：
  ssh-keygen -t ed25519 -C "你的GitHub Email" -f "$env:USERPROFILE\.ssh\id_ed25519_github"
  Get-Content "$env:USERPROFILE\.ssh\id_ed25519_github.pub" | Set-Clipboard
  → 貼到 GitHub 的 SSH keys
  Set-Service ssh-agent -StartupType Automatic
  Start-Service ssh-agent
  ssh-add "$env:USERPROFILE\.ssh\id_ed25519_github"
  ssh -T git@github.com   # 測試成功

4) 下載你的專案：
  cd $env:USERPROFILE\Documents
  git clone git@github.com:你的帳號/你的repo.git
  cd 你的repo

5) （若是 Python 專案）依 repo 說明重建環境（範例）：
  # Conda
  mamba env create -f environment.yml   # 或 conda env create -f environment.yml
  conda activate 專案環境名
  # 或 pip
  python -m venv .venv
  .\.venv\Scripts\Activate.ps1
  pip install -r requirements.txt

6) 修改檔案 → 提交 → 上傳：
  git pull --rebase
  git add -A
  git commit -m "在新電腦上的第一次修改"
  git push

---

<div style="page-break-after: always;"></div>

# Git 常用指令速查
Git 常用指令速查（Windows / PowerShell）

【檢視狀態與歷史】
  git status                     # 查看目前變更與分支狀態
  git log --oneline --graph -n 20  # 精簡歷史圖
  git show <commit>              # 看某次提交的內容與差異
  git diff                       # 查看尚未加入暫存區的差異
  git diff --staged              # 查看已加入暫存區的差異

【新增、提交、推送】
  git add -A                     # 加入全部變更
  git commit -m "訊息"            # 建立提交
  git push                       # 推送到遠端（若已設定 upstream）
  git push -u origin main        # 第一次推並設定 upstream

【抓取更新】
  git pull --rebase              # 先抓取遠端再把自己的提交接到最前（較乾淨）
  git fetch                      # 只抓提交，不自動合併

【分支操作】
  git branch                     # 列出本機分支
  git branch -vv                 # 顯示追蹤關係
  git checkout -b feature/x      # 新建並切換到分支
  git switch <branch>            # 切換分支（較新指令）
  git merge <branch>             # 合併某分支到目前分支
  git rebase <branch>            # 以 <branch> 為基底重接提交（進階）

【檔案還原與撤銷】
  git restore <檔案>             # 還原工作區檔案到 HEAD 狀態
  git restore --staged <檔案>     # 從暫存區移除（保留工作區變更）
  git reset --hard HEAD~1        # 回退 1 個提交並丟棄變更（危險）
  git clean -fd                  # 移除未追蹤檔案與資料夾（危險）

【暫存工作（stash）】
  git stash                      # 把未提交變更暫存起來
  git stash list                 # 查看暫存清單
  git stash pop                  # 套用並移除最近一次暫存

【遠端設定】
  git remote -v                  # 查看遠端列表與位址
  git remote set-url origin git@github.com:你/你的repo.git  # 改用 SSH
  git remote add origin <URL>    # 連結遠端
  git remote remove origin       # 移除遠端

【標籤（版本號）】
  git tag v1.0.0                 # 建立附註標籤（輕量）
  git tag -a v1.0.0 -m "說明"     # 建立含說明的標籤（附註）
  git push origin --tags         # 推送所有標籤
  git show v1.0.0                # 查看某標籤內容

【設定與環境】
  git config --global --list     # 檢視全域設定
  git config --global user.name "你的名字"
  git config --global user.email "你的Email"
  git config --global core.autocrlf true
  git config --global init.defaultBranch main

小提醒：
- 第一次推送用：git push -u origin main（之後只要 git push）。
- 避免用 reset --hard / clean -fd 前請先確認無需保留的變更。
- 與團隊協作時，習慣在開始工作前先：git pull --rebase。

