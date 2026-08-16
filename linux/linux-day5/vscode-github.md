# VS CodeとGitHub連携環境構築

## 目的

Ubuntu上でVS Codeを使用できる環境を構築し、GitHubと連携して学習内容を管理できる状態にする。

今回の目的：

- Ubuntuを開発環境として使用する
- VS Codeの設定をMacと同期する
- GitHubとSSH認証で連携する
- Linux学習記録をGitHubで管理する

---

# 1. UbuntuへVS Codeをインストール

## インストール確認

Ubuntu標準のaptではVS Codeパッケージが存在しなかった。

```bash
sudo apt install code
```

結果：

```
Unable to locate package code
```

そのためMicrosoft公式リポジトリを追加してインストールした。

---

## VS Codeインストール

必要パッケージをインストール。

```bash
sudo apt update

sudo apt install wget gpg apt-transport-https
```

MicrosoftのGPGキーを登録。

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg

sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
```

リポジトリ追加。

```bash
echo "deb [arch=arm64 signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list
```

インストール。

```bash
sudo apt update

sudo apt install code
```

起動確認。

```bash
code
```

---

# 2. VS Code Settings Sync設定

Macで使用しているVS Code設定をUbuntuへ同期。

同期内容：

- 設定
- キーボードショートカット
- スニペット
- タスク
- 拡張機能
- プロファイル

GitHubアカウントでVS Codeへサインイン。

認証後、同期完了。

---

# 3. VS Code拡張機能追加

## GitLens

Git履歴確認用の拡張。

できること：

- 変更者確認
- commit履歴確認
- 差分確認
- ファイル変更履歴確認

※ GitLensのアカウント登録は不要。

---

# 4. Git初期設定

Git確認。

```bash
git --version
```

結果：

```
git version 2.53.0
```

---

ユーザー設定。

```bash
git config --global user.name "nobuhitokamino"

git config --global user.email "GitHub登録メール"
```

確認。

```bash
git config --list
```

結果：

```
user.name=nobuhitokamino
user.email=xxxxx@gmail.com
```

---

# 5. GitHub SSH認証設定

## SSH鍵作成

SSHフォルダ確認。

```bash
ls -al ~/.ssh
```

GitHub用SSH鍵作成。

```bash
ssh-keygen -t ed25519 -C "GitHub登録メール"
```

作成された鍵：

```
~/.ssh/

├── id_ed25519
└── id_ed25519.pub
```

---

## 公開鍵をGitHubへ登録

公開鍵確認。

```bash
cat ~/.ssh/id_ed25519.pub
```

表示された内容をGitHubへ登録。

GitHub：

```
Settings
 ↓
SSH and GPG keys
 ↓
New SSH key
```

Title：

```
Ubuntu-study
```

として登録。

---

## SSH接続確認

```bash
ssh -T git@github.com
```

結果：

```
Hi nobuhitokamino! You've successfully authenticated,
but GitHub does not provide shell access.
```

SSH認証成功。

---

# 6. GitHubリポジトリ作成

既存の学習フォルダ：

```
infra-study

├── linux
│   ├── linux-day1
│   ├── linux-day2
│   ├── linux-day3
│   ├── linux-day4
│   ├── linux-day5
│   └── Troubleshooting
│
└── README.md
```

を管理するためGitHubに：

```
infra-study
```

リポジトリを作成。

設定：

- Public
- README追加なし
- Licenseなし

理由：

ローカル側にすでにREADMEが存在しているため。

---

# 7. ローカルリポジトリをGitHubへpush

作業場所：

```bash
cd ~/infra-study
```

Git初期化。

```bash
git init
```

GitHubとの接続設定。

```bash
git remote add origin git@github.com:nobuhitokamino/infra-study.git
```

ファイル追加。

```bash
git add .
```

commit。

```bash
git commit -m "Add Linux study notes"
```

---

## mainブランチへ変更

初期状態：

```
master
```

だったため変更。

```bash
git branch -M main
```

意味：

- `-M` → 強制的にブランチ名変更
- `main` → 変更後の名前

変更：

```
master
 ↓
main
```

---

## GitHubへpush

```bash
git push -u origin main
```

成功：

```
[new branch] main -> main

branch 'main' set up to track 'origin/main'
```

---

# 8. clone確認

別フォルダでcloneテスト。

```bash
mkdir test-clone

cd test-clone

git clone git@github.com:nobuhitokamino/infra-study.git
```

確認：

```bash
ls
```

結果：

```
infra-study
```

---

状態確認。

```bash
git status
```

結果：

```
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

GitHubとの同期確認完了。

---

# 9. 不要なclone削除

確認用clone削除。

```bash
rm -rf test-clone
```

確認：

```bash
ls
```

削除完了。

---

# 完成した環境

```
Mac
│
└── UTM
    │
    └── Ubuntu
        ├── VS Code
        ├── Git
        ├── GitHub SSH認証
        └── infra-study
            ├── Linux学習記録
            ├── AWS学習予定
            └── Terraform学習予定
```

---

# 今後のGit基本操作

## 変更確認

```bash
git status
```

## ファイル追加

```bash
git add .
```

## commit

```bash
git commit -m "変更内容"
```

## GitHubへ反映

```bash
git push
```

## GitHubから取得

```bash
git pull
```

---

# 学習ポイント

今回学んだこと：

- Ubuntuで開発環境を構築する方法
- VS Code設定同期
- Git初期設定
- SSH公開鍵認証
- GitHubとの安全な接続
- リポジトリ作成
- push / cloneの流れ
- mainブランチ管理

インフラエンジニアとして必要なLinux・Git・GitHubの基礎環境を構築できた。
