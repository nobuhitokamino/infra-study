# Linux学習 Day6
## ファイル権限管理（chmod・chown・グループ管理）

## 今日の学習内容

Linuxのファイルやディレクトリの権限管理について学習した。

今回の目的：

- chmodによる権限変更
- chownによる所有者変更
- グループ管理
- ディレクトリ権限の理解
- Permission deniedの原因調査

---

# 1. ファイル権限の確認

## ls -l

ファイルの権限や所有者を確認する。

実行：

    ls -l

例：

    -rw-rw-r-- 1 nobuhito nobuhito 0 Aug 17 20:43 test.txt

権限部分：

    -rw-rw-r--
    ││ │ │
    ││ │ └── その他ユーザー
    ││ └──── グループ
    │└────── 所有者
    └─────── ファイル種別

---

## 権限の意味

|記号|意味|
|-|-|
|r|read（読み取り）|
|w|write（書き込み）|
|x|execute（実行）|
|-|権限なし|

---

# 2. chmodによる権限変更

## chmodとは

ファイルやディレクトリの権限を変更するコマンド。

書式：

    chmod 権限 ファイル名

---

## 数字指定

権限は数字で表現できる。

|権限|数字|
|-|-|
|r|4|
|w|2|
|x|1|

例：

    rw- = 4+2 = 6

    r-x = 4+1 = 5

---

## chmod 600

実行：

    chmod 600 test.txt

変更前：

    -rw-rw-r--

変更後：

    -rw-------

意味：

|対象|権限|
|-|-|
|所有者|rw-|
|グループ|---|
|その他|---|

所有者だけが読み書きできる状態。

---

# 3. chownによる所有者変更

## chownとは

ファイルやディレクトリの所有者を変更するコマンド。

書式：

    sudo chown ユーザー ファイル

---

実験：

変更前：

    -rw------- 1 nobuhito nobuhito test.txt

実行：

    sudo chown linux-test test.txt

変更後：

    -rw------- 1 linux-test nobuhito test.txt

所有者が変更された。

    nobuhito
        ↓
    linux-test

---

# 4. Permission deniedの原因調査

所有者をlinux-testへ変更したが、最初はアクセスできなかった。

原因調査：

    namei -l /home/nobuhito/infra-study/linux/linux-day6/test.txt

結果：

    drwxr-x--- nobuhito nobuhito nobuhito
    drwxrwxr-x nobuhito nobuhito infra-study
    drwxrwxr-x nobuhito nobuhito linux
    drwxrwxr-x nobuhito nobuhito linux-day6
    -rw------- linux-test nobuhito test.txt

---

## 原因

ファイル自体：

    -rw------- linux-test

は問題なかった。

しかし、

    /home/nobuhito

の権限が：

    drwxr-x---

だった。

linux-testはnobuhitoグループに所属していなかったため、

    その他ユーザー
    ---

として扱われ、ディレクトリへ入ることができなかった。

---

# 5. グループ追加

linux-testをnobuhitoグループへ追加した。

確認：

    groups linux-test

変更前：

    linux-test sudo users

---

追加：

    sudo usermod -aG nobuhito linux-test

確認：

    groups linux-test

結果：

    linux-test sudo users nobuhito

---

再ログイン後：

    id

結果：

    uid=1002(linux-test)
    groups=1002(linux-test),27(sudo),100(users),1000(nobuhito)

nobuhitoグループへの追加を確認。

---

# 6. linux-testでアクセス確認

linux-testへ切り替え：

    su - linux-test

確認：

    whoami

結果：

    linux-test

---

ファイル確認：

    ls -l /home/nobuhito/infra-study/linux/linux-day6/test.txt

結果：

    -rw------- 1 linux-test nobuhito test.txt

---

書き込みテスト：

    echo "linux-test user" >> /home/nobuhito/infra-study/linux/linux-day6/test.txt

確認：

    cat /home/nobuhito/infra-study/linux/linux-day6/test.txt

結果：

    linux-test user

書き込み成功。

---

# 今日の重要ポイント

## ファイル権限だけではアクセスできない

Linuxではファイルだけではなく、そこまでのパスにあるディレクトリ権限も確認される。

例：

    /home
      ↓
    /home/nobuhito
      ↓
    infra-study
      ↓
    linux
      ↓
    linux-day6
      ↓
    test.txt

すべてのディレクトリを通過できる必要がある。

---

## ディレクトリのx権限

ファイルの場合：

    x = 実行

ディレクトリの場合：

    x = 中へ入る権限

になる。

---

# 今日覚えたコマンド

|コマンド|用途|
|-|-|
|ls -l|権限確認|
|chmod|権限変更|
|chown|所有者変更|
|chgrp|グループ変更|
|id|ユーザー・グループ確認|
|groups|所属グループ確認|
|usermod -aG|グループ追加|
|namei -l|パスごとの権限確認|

---

# Day6まとめ

今回はchmod、chownを使った権限管理を実践した。

特に、

「所有者を変更したのにPermission deniedになる」

というLinuxでよくある問題を経験した。

原因はファイルではなく、親ディレクトリの権限だった。

Linuxの権限トラブル調査では、

1. whoamiでユーザー確認
2. id / groupsで所属グループ確認
3. ls -lでファイル権限確認
4. namei -lでパス権限確認

という流れで原因調査できることを学んだ。
