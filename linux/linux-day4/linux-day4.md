# Linux学習 Day4 まとめ

## 学習テーマ

- プロセス管理の基本
- PIDとPPIDの理解
- psコマンドによるプロセス確認
- pstreeによるプロセス階層確認
- killによるプロセス終了
- topによるリソース確認
- systemdサービス管理
- journalctlによるログ確認

---

# 1. プロセスとは

## プロセス（Process）

Linuxで動作しているプログラムの実行単位。

例:

- bash
- ターミナル
- Webサーバー
- cron
- rsyslog

など、Linux上で動いているものは全てプロセスとして管理される。

---

# 2. PIDとPPID

## PID（Process ID）

プロセスを識別する番号。

確認例:

```bash
ps aux
```

表示例:

```
nobuhito  3650  3608  /usr/bin/bash
```

この場合:

```
PID = 3650
```

bashプロセスの番号。

---

## PPID（Parent Process ID）

親プロセスのID。

確認:

```bash
ps -fp PID
```

例:

```bash
ps -fp 3650
```

結果:

```
UID       PID   PPID
nobuhito  3650  3608
```

意味:

- PID 3650の親プロセスはPID 3608

---

# 3. psコマンド

プロセスを確認するコマンド。

## ps

現在のプロセスを表示。

```bash
ps
```

---

## ps aux

全ユーザーのプロセスを詳細表示。

```bash
ps aux
```

主な項目:

| 項目 | 内容 |
|---|---|
| USER | 実行ユーザー |
| PID | プロセスID |
| %CPU | CPU使用率 |
| %MEM | メモリ使用率 |
| STAT | 状態 |
| COMMAND | 実行コマンド |

---

## grepと組み合わせる

特定プロセスを検索。

例:

```bash
ps aux | grep bash
```

意味:

- 全プロセスからbashを検索

---

# 4. pstreeコマンド

プロセスを親子関係で表示する。

```bash
pstree
```

PID付き:

```bash
pstree -p
```

例:

```
ptyxis-agent(3608)
 └─bash(3650)
    └─pstree(4221)
```

意味:

```
ターミナル
 ↓
bash
 ↓
実行したコマンド
```

というプロセスの親子関係が確認できる。

---

# 5. バックグラウンドプロセス

## &

コマンドをバックグラウンド実行する。

例:

```bash
sleep 300 &
```

意味:

- sleepを実行
- 端末操作は継続可能

結果:

```
[1] 4116
```

この番号:

```
4116 = PID
```

---

# 6. killコマンド

プロセスを終了する。

基本:

```bash
kill PID
```

例:

```bash
kill 4116
```

意味:

```
PID 4116のプロセスを終了
```

確認:

```bash
ps aux | grep sleep
```

終了後:

```
sleep
```

が表示されなくなる。

---

# 7. topコマンド

リアルタイムでシステム状態を確認する。

```bash
top
```

確認できる内容:

- CPU使用率
- メモリ使用率
- 実行中プロセス
- Load Average

---

## topで見るポイント

### CPU

例:

```
%Cpu(s): 7.6 us, 0.7 sy, 91.2 id
```

意味:

|項目|意味|
|-|-|
|us|ユーザープロセスCPU|
|sy|カーネル処理CPU|
|id|CPU空き|

---

### メモリ

例:

```
MiB Mem:
total
used
free
```

確認:

- メモリ不足がないか
- 使用量が異常に高いプロセスがないか

---

# 8. systemdとは

Linuxのサービス管理システム。

役割:

- サービス起動
- サービス停止
- 自動起動設定
- ログ管理

---

# 9. systemctlコマンド

systemdを操作するコマンド。

## サービス状態確認

```bash
systemctl status サービス名
```

例:

```bash
systemctl status cron
```

---

状態:

```
Active: active (running)
```

意味:

- サービス稼働中

---

状態:

```
Active: inactive (dead)
```

意味:

- サービス停止中

---

# 10. サービス停止・起動

## 停止

```bash
sudo systemctl stop cron
```

確認:

```bash
systemctl status cron
```

結果:

```
Active: inactive (dead)
```

---

## 起動

```bash
sudo systemctl start cron
```

確認:

```bash
systemctl status cron
```

結果:

```
Active: active (running)
```

---

# 11. cronサービス

cronは定期的に処理を実行するサービス。

例:

- バックアップ
- ログ整理
- 定期実行処理

確認:

```bash
systemctl status cron
```

---

# 12. journalctl

## journalctlとは

systemdが管理するログを見るコマンド。

認識:

```
journalctl = systemd journalのログ確認コマンド
```

---

## 最新ログ表示

```bash
journalctl -n 20
```

意味:

- 最新20件のログ表示

---

## サービス単位でログ確認

```bash
journalctl -u サービス名
```

例:

```bash
journalctl -u cron
```

意味:

- cronサービスのログを見る

---

## 最新N件

```bash
journalctl -u cron -n 10
```

意味:

- cronログを最新10件表示

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

# 13. journalctlで確認できる内容

例:

サービス起動:

```
Started cron.service
```

サービス停止:

```
Stopped cron.service
```

サービス再起動:

```
Starting cron.service
```

など。

---

# 14. 障害調査で使う流れ

## ① プロセス確認

```bash
ps aux
```

または

```bash
top
```

↓

## ② サービス状態確認

```bash
systemctl status サービス名
```

↓

## ③ ログ確認

```bash
journalctl -u サービス名
```

---

# 15. Day4で覚えた重要コマンド一覧

| コマンド | 内容 |
|---|---|
| ps aux | 全プロセス確認 |
| ps -fp PID | PID詳細確認 |
| pstree -p | プロセス階層確認 |
| sleep 300 & | バックグラウンド実行 |
| kill PID | プロセス終了 |
| top | CPU・メモリ・プロセス確認 |
| systemctl status | サービス状態確認 |
| systemctl start | サービス起動 |
| systemctl stop | サービス停止 |
| journalctl | systemdログ確認 |
| journalctl -u | サービスログ確認 |
| journalctl -f | ログリアルタイム監視 |

---

# Day4の理解ポイント

- Linuxでは全ての処理がプロセスとして管理される
- PIDでプロセスを識別する
- PPIDを見ることで親子関係を理解できる
- psで現在のプロセスを確認できる
- pstreeでプロセスの構造を理解できる
- killで不要なプロセスを終了できる
- topでシステム負荷を確認できる
- systemctlでサービスを管理できる
- journalctlでsystemd管理ログを確認できる
- 障害対応では「状態確認 → ログ確認」の流れが重要
