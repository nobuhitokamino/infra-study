# Linux学習 Day1：Linux環境確認・ファイルシステム・権限管理

## 学習目的

LinuC 101/102取得レベルのLinux操作スキル習得に向けて、Linuxの基本構造と操作方法を理解する。

今回の学習内容：

- Linux環境確認
- Linuxファイルシステム構造
- 主要ディレクトリの役割
- ユーザー・グループ管理の基礎
- ファイル操作
- Linuxパーミッション
- chmod
- umask

---

# 1. Linux環境確認

## OS情報確認

### コマンド

```bash
cat /etc/os-release
```

### 確認できること

- Linuxディストリビューション
- OSバージョン
- ベースとなるOS

今回の環境：

```
Ubuntu 26.04 LTS
```

---

## Kernel情報確認

### コマンド

```bash
uname -a
```

### 確認できること

- Linux Kernelバージョン
- CPUアーキテクチャ
- OS情報

今回：

```
aarch64
```

Apple Silicon Mac上のARM64 Ubuntu環境であることを確認。

---

## 現在ログインしているユーザー確認

### コマンド

```bash
whoami
```

結果：

```
nobuhito
```

現在操作しているユーザーを確認。

---

## 現在位置確認

### コマンド

```bash
pwd
```

結果：

```
/home/nobuhito
```

現在いるディレクトリを確認。

---

# 2. Linuxファイルシステム構造

Linuxではすべてのディレクトリが `/`（ルートディレクトリ）から始まる。

Windowsのような `C:\` や `D:\` ではなく、1つのツリー構造になっている。

## Linuxディレクトリ構造

```
/
├── home  ユーザーのホームディレクトリ
├── etc   システム設定ファイル
├── var   ログなど変化するデータ
├── usr   コマンド・アプリケーション
├── proc  Kernel情報
├── sys   ハードウェア情報
├── dev   デバイスファイル
├── tmp   一時ファイル
└── root  rootユーザーのホーム
```

---

# 3. 主要ディレクトリの役割

## /home

一般ユーザーのホームディレクトリ。

例：

```
/home/nobuhito
```

役割：

- 個人ファイル保存
- 作業ディレクトリ
- ユーザー設定

---

## /root

rootユーザー専用のホームディレクトリ。

注意：

```
/
```

と

```
/root
```

は別物。

---

## /etc

Linuxシステムの設定ファイルを保存する場所。

例：

```
/etc/ssh
/etc/passwd
/etc/fstab
```

サーバー管理では頻繁に確認する場所。

---

## /var

変化するデータを保存する場所。

例：

```
/var/log
```

ログファイル保存場所。

障害調査では重要。

---

## /proc

Kernelが提供する仮想ファイルシステム。

通常のファイルではなく、システム情報を参照するための仕組み。

例：

CPU情報：

```bash
cat /proc/cpuinfo
```

メモリ情報：

```bash
cat /proc/meminfo
```

プロセス情報：

```
/proc/PID
```

---

## /dev

デバイスファイルを管理する場所。

例：

```
/dev/vda
/dev/null
```

Linuxではデバイスをファイルとして扱う。

---

# 4. /etcの確認

## コマンド

```bash
ls /etc
```

確認内容：

Linuxシステムやサービスの設定ファイルが大量に存在する。

---

## SSH設定

確認：

```bash
ls /etc/ssh
```

SSHクライアント設定などを確認。

SSHには以下がある。

### SSHクライアント

接続する側。

設定：

```
/etc/ssh/ssh_config
```

---

### SSHサーバー

接続される側。

設定：

```
/etc/ssh/sshd_config
```

---

# 5. ユーザー情報確認

## /etc/passwd

確認：

```bash
cat /etc/passwd | grep nobuhito
```

結果：

```
nobuhito:x:1000:1000:nobuhito:/home/nobuhito:/bin/bash
```

意味：

|項目|意味|
|-|-|
|nobuhito|ユーザー名|
|x|パスワード情報はshadow管理|
|1000|UID|
|1000|GID|
|/home/nobuhito|ホームディレクトリ|
|/bin/bash|ログインシェル|

---

# 6. ログ確認

## /var/log

確認：

```bash
ls /var/log
```

主なログ：

```
auth.log
syslog
kern.log
cloud-init.log
cloud-init-output.log
```

用途：

- エラー調査
- システム状態確認
- 起動トラブル調査

AWS EC2では：

```
EC2障害発生
↓
ログ確認
↓
原因調査
```

という流れになる。

---

# 7. CPU・メモリ確認

## CPU情報

```bash
cat /proc/cpuinfo
```

確認できること：

- CPU情報
- アーキテクチャ

---

## メモリ情報

```bash
cat /proc/meminfo
```

確認できること：

- 総メモリ
- 空きメモリ
- キャッシュ情報

---

# 8. ディスク構造確認

## コマンド

```bash
lsblk
```

結果：

```
vda
├─vda1
└─vda2
```

Linuxのディスク構造：

```
ディスク
 ↓
パーティション
 ↓
ファイルシステム
 ↓
マウント
 ↓
利用
```

今回：

```
/dev/vda2
 ↓
/
```

として利用。

AWSの場合：

```
EBS
 ↓
/dev/xvda
 ↓
Linuxファイルシステム
```

という構成になる。

---

# 9. ファイル操作

## 作業ディレクトリ作成

```bash
mkdir linux-day1
cd linux-day1
```

---

## ファイル作成

```bash
touch server.txt client.txt network.txt
```

---

## ファイルへ書き込み

```bash
echo "Linux Server" > server.txt
```

`>` はリダイレクト。

標準出力をファイルへ書き込む。

---

## 内容確認

```bash
cat server.txt
```

---

# 10. Linuxパーミッション

確認：

```bash
ls -l
```

例：

```
-rw-rw-r--
```

分解：

```
- rw- rw- r--
│ │   │   │
│ │   │   └ その他ユーザー
│ │   └──── グループ
│ └──────── 所有者
└────────── ファイル種別
```

---

## 権限種類

|記号|意味|
|-|-|
|r|read（読み取り）|
|w|write（書き込み）|
|x|execute（実行）|

---

# 11. chmod

権限は数字でも表現できる。

|権限|数字|
|-|-:|
|r|4|
|w|2|
|x|1|

---

## chmod 600

```bash
chmod 600 server.txt
```

結果：

```
rw-------
```

意味：

- 所有者：読み書き可能
- グループ：不可
- その他：不可

---

## chmod 644

```bash
chmod 644 client.txt
```

結果：

```
rw-r--r--
```

意味：

- 所有者：読み書き
- グループ：読み取り
- その他：読み取り

---

## chmod 755

```bash
chmod 755 network.txt
```

結果：

```
rwxr-xr-x
```

意味：

- 所有者：読み書き実行
- グループ：読み取り実行
- その他：読み取り実行

---

# 12. ユーザー・グループ確認

## groups

```bash
groups
```

所属グループを確認。

例：

```
nobuhito sudo
```

sudoグループ所属の場合、管理者権限を利用可能。

---

## id

```bash
id
```

確認：

- UID
- GID
- 所属グループ

例：

```
uid=1000(nobuhito)
gid=1000(nobuhito)
```

Linuxではユーザーを名前だけではなくIDでも管理している。

---

# 13. ディレクトリ権限

作成：

```bash
mkdir backup
```

確認：

```bash
ls -ld backup
```

例：

```
drwxrwxr-x
```

ディレクトリの場合：

|権限|意味|
|-|-|
|r|中身を見る|
|w|作成・削除|
|x|ディレクトリへ移動|

---

# 14. cpコマンド

コピー：

```bash
cp -i server.txt backup/
```

`-i`

interactive

上書き前に確認する安全オプション。

---

# 15. umask

確認：

```bash
umask
```

結果：

```
0002
```

umaskは新規作成ファイル・ディレクトリの初期権限を決める。

---

## ファイルの場合

基本権限：

```
666
```

計算：

```
666 - 002 = 664
```

結果：

```
rw-rw-r--
```

---

## ディレクトリの場合

基本権限：

```
777
```

計算：

```
777 - 002 = 775
```

結果：

```
rwxrwxr-x
```

---

# Day1確認問題まとめ

## Q1

/etcには何が置かれている？

回答：

システムの設定ファイル。

---

## Q2

/var/logは何のため？

回答：

システムやアプリケーションのログ保存。
エラー原因調査に利用。

---

## Q3

権限：

```
-rwxr-xr--
```

数字表記：

```
754
```

---

## Q4

chmod 640 file.txt の意味：

```
rw-r-----
```

所有者：

読み書き可能

グループ：

読み取り可能

その他：

権限なし

---

## Q5

/procとは？

回答：

Kernelが提供する仮想ファイルシステム。

---

## Q6

UID 1000とは？

回答：

ユーザーID。

Linux内部ではユーザーをIDで管理している。

---

## Q7

umask 002の場合のディレクトリ権限：

```
777 - 002 = 775
```

---

# Day1まとめ

Linuxは単なるコマンド操作ではなく、

```
ユーザー
 ↓
グループ
 ↓
所有権
 ↓
パーミッション
 ↓
アクセス制御
```

で管理されるOSであることを理解した。

また、Linuxの基本構造：

```
/
├── etc  → 設定
├── var  → ログ
├── home → ユーザー
├── proc → Kernel情報
└── dev  → デバイス
```

を理解した。

AWSでのEC2運用でも、

- SSH接続
- 設定変更
- ログ調査
- 権限エラー対応
- 障害調査

にLinuxの基礎知識が必要になる。

---
