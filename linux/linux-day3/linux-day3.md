# Linux学習 Day3 まとめ

## 学習テーマ

- ファイル操作コマンドの復習
- ログファイルの確認
- grepによる検索
- tailによるログ監視
- Linuxログ管理の基本

---

# 1. ファイル操作復習

## cp

ファイルをコピーするコマンド。

```bash
cp コピー元 コピー先
```

使用例:

```bash
cp test.txt backup.txt
```

- test.txtをbackup.txtとしてコピーする
- 元ファイルは残る

---

# 2. grep

文字列を検索するコマンド。

```bash
grep 検索文字 ファイル名
```

使用例:

```bash
grep error app.log
```

意味:

- app.log内からerrorという文字列を検索する

よく使うオプション:

| オプション | 内容 |
|---|---|
| -i | 大文字小文字を区別しない |
| -n | 行番号を表示 |
| -r | ディレクトリ内を再帰検索 |

---

# 3. tail

ファイルの末尾を表示するコマンド。

```bash
tail ファイル名
```

ログ確認では以下をよく使う。

```bash
tail -f ファイル名
```

## tail -f

リアルタイムでログを監視する。

例:

```bash
tail -f /var/log/syslog
```

ログが追加されるたびに表示される。

終了:

```
Ctrl + C
```

---

# 4. Linuxログ確認

Linuxではシステムやサービスの動作記録をログとして保存している。

主なログ:

| ファイル | 内容 |
|---|---|
| /var/log/syslog | システム全般のログ |
| /var/log/auth.log | 認証・sudoなどのログ |

確認:

```bash
cat /var/log/syslog
```

または

```bash
less /var/log/syslog
```

---

# 5. systemdログ確認

systemd管理のログを見るにはjournalctlを使う。

## journalctlとは

systemdのログ管理システム
（systemd journal）のログを見るコマンド。

基本:

```bash
journalctl
```

---

## 最新ログ表示

```bash
journalctl -n 20
```

意味:

- 最新20件のログを表示

---

## サービス単位で確認

```bash
journalctl -u サービス名
```

例:

```bash
journalctl -u rsyslog
```

意味:

- rsyslogサービスのログだけ表示

---

## リアルタイム監視

```bash
journalctl -f
```

意味:

- 新しいログをリアルタイム表示

終了:

```
Ctrl + C
```

---

# 6. rsyslog確認

rsyslogはLinuxのログ管理サービス。

確認:

```bash
systemctl status rsyslog
```

結果:

```
Active: active (running)
```

の場合、サービスが正常稼働中。

---

再起動:

```bash
sudo systemctl restart rsyslog
```

---

ログ確認:

```bash
journalctl -u rsyslog
```

---

# 7. NetworkManager確認

NetworkManagerはネットワーク接続を管理するサービス。

確認:

```bash
systemctl status NetworkManager
```

状態:

```
Active: active (running)
```

なら正常。

---

# 8. Day3で覚えた重要コマンド一覧

| コマンド | 内容 |
|---|---|
| cp | ファイルコピー |
| mv | ファイル移動 |
| find | ファイル検索 |
| grep | 文字列検索 |
| tail | ファイル末尾表示 |
| tail -f | ログリアルタイム監視 |
| systemctl status | サービス状態確認 |
| journalctl | systemdログ確認 |

---

# Day3の理解ポイント

- Linuxではログ確認が障害調査の基本になる
- `/var/log`にはシステムログが保存される
- `systemctl`でサービス状態を確認する
- `journalctl`でsystemd管理のログを確認できる
- `tail -f`でリアルタイムログ監視ができる
