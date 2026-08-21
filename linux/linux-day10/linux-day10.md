# Linux学習 Day10 - パッケージ管理（apt / dpkg）

## 今日の学習内容

Day10では、Ubuntuにおけるパッケージ管理について学習した。

Linuxでは、ソフトウェアのインストール・更新・削除をパッケージ管理システムで行う。

Ubuntuでは主に以下を使用する。

- apt
  - パッケージ管理を行う高レベルツール
  - 依存関係を自動解決する

- dpkg
  - `.deb`パッケージを管理する低レベルツール
  - 実際のインストール処理を担当する


---

# Ubuntuのパッケージ管理の流れ

    リポジトリ
        ↓
    apt update
        ↓
    パッケージ情報取得
        ↓
    apt install
        ↓
    依存関係解決
        ↓
    dpkgがインストール
        ↓
    systemctlでサービス管理


---

# Ubuntuバージョン確認

## コマンド

    cat /etc/os-release

結果：

    PRETTY_NAME="Ubuntu 26.04 LTS"

確認内容：

- Ubuntu 26.04 LTSを使用
- Debian系Linux
- apt / dpkgを利用する環境


---

# apt update

## コマンド

    sudo apt update

役割：

    パッケージ一覧を最新化する

注意：

apt updateはソフトウェア自体を更新するコマンドではない。

処理：

    Ubuntuリポジトリ
        ↓
    最新パッケージ情報取得
        ↓
    更新可能なパッケージを確認


---

# 更新可能パッケージ確認

## コマンド

    apt list --upgradable

役割：

    現在インストールされているパッケージの中で
    更新可能なものを確認する


---

# apt upgrade

## コマンド

    sudo apt upgrade

役割：

    インストール済みパッケージを最新バージョンへ更新する


apt updateとの違い：

    apt update

    → パッケージ情報を更新する


    apt upgrade

    → 実際のソフトウェアを更新する


---

# phased updates（段階的アップデート）

アップグレード時に表示された：

    Not upgrading yet due to phasing

これはエラーではない。

Ubuntuでは安全のためアップデートを段階的に配布している。

    アップデート公開
        ↓
    一部ユーザーへ配布
        ↓
    問題確認
        ↓
    全ユーザーへ配布


---

# treeパッケージ確認

確認コマンド：

    tree --version

結果：

    tree v2.3.1

すでにインストール済みだった。


---

# dpkgでインストール確認

## コマンド

    dpkg -l | grep tree

結果：

    ii  tree  2.3.1-1  arm64

確認：

    ii

はインストール済みを意味する。

つまり：

    treeパッケージは正常にインストール済み


---

# aptとdpkgの関係

    apt

    ↓

    依存関係を解決する便利な管理ツール

    ↓

    dpkg

    ↓

    .debパッケージを実際に処理

    ↓

    インストール完了


---

# nginxパッケージ確認

Day9でインストールしたnginxを確認した。

## コマンド

    dpkg -l | grep nginx

結果：

    ii  nginx
    ii  nginx-common


確認内容：

- nginx本体がインストール済み
- nginx-commonもインストール済み


---

# apt show

## コマンド

    apt show nginx

役割：

    パッケージ詳細情報を確認する


確認できる情報：

- パッケージ名
- バージョン
- 依存関係
- サイズ
- 取得元リポジトリ
- 説明


---

# nginxの依存関係

例：

    Depends:
    libc6
    libcrypt1
    libpcre2-8-0
    libssl3t64
    zlib1g


意味：

nginxは複数のライブラリに依存して動作する。

aptは依存関係を自動的に解決してくれる。


---

# リポジトリ確認

Ubuntu 26.04では以下を使用している。

    /etc/apt/sources.list.d/ubuntu.sources


確認：

    cat /etc/apt/sources.list.d/ubuntu.sources


---

# リポジトリ設定

## URIs

    http://jp.archive.ubuntu.com/ubuntu/

意味：

パッケージ取得先。

今回は日本のUbuntuミラーサーバーを利用。


---

## Suites

    resolute
    resolute-updates
    resolute-backports
    resolute-security


意味：

Ubuntuのリリースや更新種類。


---

## Components

    main
    restricted
    universe
    multiverse


意味：

Ubuntuのパッケージ分類。


|種類|意味|
|-|-|
|main|Ubuntu公式サポート|
|restricted|制限付きソフトウェア|
|universe|コミュニティ管理|
|multiverse|ライセンス制限あり|


---

# Signed-By

設定：

    /usr/share/keyrings/ubuntu-archive-keyring.gpg


意味：

Ubuntu公式パッケージか確認するための署名鍵。


流れ：

    リポジトリ
        ↓
    パッケージ取得
        ↓
    GPG署名確認
        ↓
    安全ならインストール


---

# 今日覚えたコマンドまとめ

パッケージ情報更新：

    sudo apt update


パッケージ更新：

    sudo apt upgrade


更新対象確認：

    apt list --upgradable


インストール：

    sudo apt install パッケージ名


詳細確認：

    apt show パッケージ名


インストール済み確認：

    dpkg -l


特定パッケージ確認：

    dpkg -l | grep パッケージ名


---

# Day10の学び

Linuxのパッケージ管理の流れを理解した。

    リポジトリ
        ↓
    apt update
        ↓
    apt install
        ↓
    dpkg
        ↓
    systemctl


この流れはAWSなどでLinuxサーバーを構築するときにも使用する。

EC2構築時の：

    apt install

    dnf install

などの意味が理解できるようになった。

Day10では、Linuxサーバー管理に必要なパッケージ管理の基礎を習得した。
