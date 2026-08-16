# Linux学習 Day5 ユーザー管理・グループ管理・sudo

## 今日の学習内容

Day5ではLinuxにおけるユーザー管理、グループ管理、sudo権限について学習した。

Linuxでは「誰が操作しているか」「何を操作できるか」をユーザー・グループ・権限によって管理している。

インフラ運用ではサーバーにログインするユーザー管理や権限設定を行うため、重要な基礎知識になる。

---

# 1. 現在のユーザー確認

## whoami

現在ログインしているユーザーを確認するコマンド。

```bash
whoami
```

実行結果：

```text
nobuhito
```

現在操作しているユーザーが `nobuhito` であることを確認した。

---

## id

ユーザーIDや所属グループを確認するコマンド。

```bash
id
```

実行結果：

```text
uid=1000(nobuhito)
gid=1000(nobuhito)
groups=1000(nobuhito),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),111(lpadmin),114(lxd)
```

確認ポイント：

|項目|意味|
|-|-|
|uid|ユーザーID|
|gid|グループID|
|groups|所属グループ|

今回 `nobuhito` は `sudo` グループに所属しているため、管理者権限を利用できる。

---

# 2. ユーザー作成

## adduser

ユーザーを追加する。

```bash
sudo adduser linux-test
```

作成後、ユーザー情報を確認。

```bash
cat /etc/passwd | grep linux-test
```

結果：

```text
linux-test:x:1002:1002:,,,:/home/linux-test:/bin/bash
```

---

## /etc/passwdの構造

`/etc/passwd` にはユーザー情報が保存されている。

形式：

```text
ユーザー名:パスワード情報:UID:GID:コメント:ホームディレクトリ:ログインシェル
```

今回の場合：

|項目|内容|
|-|-|
|ユーザー名|linux-test|
|パスワード情報|x|
|UID|1002|
|GID|1002|
|ホームディレクトリ|/home/linux-test|
|ログインシェル|/bin/bash|

---

# 3. ホームディレクトリと権限確認

確認：

```bash
ls -ld /home/linux-test
```

結果：

```text
drwxr-x--- 2 linux-test linux-test 4096 Aug 16 16:39 /home/linux-test
```

権限：

```text
d rwx r-x ---
```

意味：

|対象|権限|内容|
|-|-|-|
|所有者|rwx|読み取り・書き込み・実行可能|
|グループ|r-x|読み取り・実行可能|
|その他|---|アクセス不可|

ユーザーごとにホームディレクトリが分けられ、他ユーザーから保護されていることを確認した。

---

# 4. ユーザー切替

## su -

ユーザーを切り替える。

```bash
su - linux-test
```

確認：

```bash
whoami
```

結果：

```text
linux-test
```

また、

```bash
pwd
```

結果：

```text
/home/linux-test
```

となり、ユーザーだけではなくホームディレクトリや環境も切り替わっていることを確認した。

---

# 5. sudo権限確認

作成直後の `linux-test` ではsudo権限がなかった。

確認：

```bash
sudo ls /root
```

結果：

```text
sudo: I'm sorry linux-test. I'm afraid I can't do that
```

sudoグループに所属していないため、管理者操作ができないことを確認した。

---

# 6. sudo権限を付与

nobuhitoユーザーに戻り、linux-testをsudoグループへ追加した。

```bash
sudo usermod -aG sudo linux-test
```

意味：

|オプション|意味|
|-|-|
|-a|既存グループを維持して追加|
|-G|補助グループ指定|

確認：

```bash
groups linux-test
```

結果：

```text
linux-test : linux-test sudo users
```

sudoグループが追加されたことを確認した。

---

# 7. sudo利用確認

ログインし直した後、

```bash
id
```

確認：

```text
uid=1002(linux-test)
gid=1002(linux-test)
groups=1002(linux-test),27(sudo),100(users)
```

sudoグループ所属を確認。

その後：

```bash
sudo ls /root
```

実行。

結果：

```text
snap
```

sudo権限でrootディレクトリを確認できた。

---

# 8. sudoersの仕組み

sudoの設定ファイル：

```text
/etc/sudoers
```

確認：

```bash
sudo cat /etc/sudoers | grep sudo
```

重要な設定：

```text
%sudo ALL=(ALL:ALL) ALL
```

意味：

- `%sudo`
  - sudoグループを指定

- `ALL=(ALL:ALL)`
  - どのユーザー・グループとして実行可能

- `ALL`
  - すべてのコマンドを許可

つまり、

「sudoグループに所属するユーザーは、すべてのコマンドをroot権限で実行できる」

という設定。

---

# 9. /etc/sudoers.d

確認：

```bash
ls -l /etc/sudoers.d/
```

結果：

```text
README
```

`/etc/sudoers.d/`には追加のsudo設定を配置できる。

実務では `/etc/sudoers` を直接編集せず、

```text
/etc/sudoers.d/
```

配下にユーザーや用途ごとの設定ファイルを作成して管理することが多い。

---

# 今日覚えた重要コマンド

|コマンド|用途|
|-|-|
|whoami|現在ユーザー確認|
|id|UID/GID/所属グループ確認|
|adduser|ユーザー追加|
|passwd|パスワード変更|
|su -|ユーザー切替|
|groups|所属グループ確認|
|usermod|ユーザー設定変更|
|sudo|管理者権限で実行|
|visudo|sudo設定編集|

---

# 今日の学び・気づき

Linuxではユーザー・グループ・権限によってアクセス制御を行っている。

今回、実際にユーザーを作成し、sudo権限を追加することで、

```
ユーザー作成
↓
グループ追加
↓
sudoers設定適用
↓
root権限で操作可能
```

というLinuxの権限管理の流れを理解できた。

AWS EC2などのサーバー運用でも、

- 開発者用ユーザー作成
- sudo権限付与
- root直接利用を避ける
- 必要最低限の権限だけ与える

という考え方につながる。

また、AWS IAMの権限管理も同じく「最小権限の原則」が重要であり、Linuxのsudo管理と共通する考え方だと理解した。
