# Linux Day7 学習記録
## systemd / systemctl / サービス管理

---

## 1. 今日の学習テーマ

Day7では、Linuxのサービス管理を行う **systemd** と、その操作コマンドである **systemctl** を学習した。

インフラエンジニアとして、Linuxサーバー上で動作しているサービスの

- 起動
- 停止
- 再起動
- 状態確認
- 自動起動設定
- ログ確認
- 依存関係確認
- 障害確認

を行うために重要な知識。

---

## 2. systemdとは

`systemd` はLinuxシステムの起動処理やサービス管理を行う仕組み。

Ubuntuでは、基本的にPID 1としてsystemdが動作している。

確認コマンド：

    ps -p 1 -o pid,comm,args

実行結果：

    PID COMMAND         COMMAND
      1 systemd         /usr/lib/systemd/systemd --switched-root --system --deserialize=52 splash

### 確認できたこと

PID 1で `systemd` が動作している。

LinuxではPID 1はシステム全体の起動やサービス管理において重要なプロセス。

---

## 3. systemctl status

systemd全体の状態を確認：

    systemctl status

今回の結果：

    State: running
    Units: 607 loaded
    Jobs: 0 queued
    Failed: 0 units

### ポイント

`systemctl status` では、systemd全体の状態を確認できる。

特に重要なのは、

    State: running

システムが正常に稼働していることを示す。

また、

    Failed: 0 units

から、systemdが管理しているユニットに失敗状態のものがないことを確認できる。

---

## 4. chronyサービスの確認

時刻同期サービスである `chrony` の状態を確認した。

    systemctl status chrony

結果：

    chrony.service - chrony, an NTP client/server
    Active: active (running)

### ポイント

`Active: active (running)` は、

> chronyサービスが現在起動していて、正常に動作している

という意味。

---

## 5. is-active

サービスが現在動作しているか確認する。

    systemctl is-active chrony

結果：

    active

### 意味

    active

→ 現在サービスが起動している。

---

## 6. is-enabled

サービスがOS起動時に自動起動する設定になっているか確認する。

    systemctl is-enabled chrony

結果：

    enabled

### 意味

    enabled

→ 次回Linuxを起動したとき、自動的にchronyが起動する設定。

---

## 7. activeとenabledの違い

ここは非常に重要。

### active

    systemctl is-active chrony

    active

これは、

> 今現在、サービスが起動しているか

を表す。

### enabled

    systemctl is-enabled chrony

    enabled

これは、

> Linux起動時に、そのサービスを自動起動する設定になっているか

を表す。

つまり、

    active  = 今どうなっているか
    enabled = 次回以降の起動時にどうするか

と考える。

---

## 8. running中のサービス一覧

現在起動しているサービスを確認：

    systemctl list-units --type=service --state=running

今回、以下のようなサービスが確認できた。

    accounts-daemon.service
    avahi-daemon.service
    chrony.service
    cron.service
    dbus.service
    gdm.service
    NetworkManager.service
    rsyslog.service
    systemd-journald.service
    systemd-logind.service
    systemd-resolved.service
    keyd.service
    ...

### ポイント

このコマンドでは、

> 現在running状態のサービス

を一覧で確認できる。

---

## 9. keydサービスを停止

キーボード設定で使用していた `keyd` を停止した。

    sudo systemctl stop keyd

確認：

    systemctl is-active keyd

結果：

    failed

さらに、

    systemctl status keyd

で状態を確認した。

---

## 10. systemctl stop

    sudo systemctl stop nginx

サービスを停止する。

例えば、

    sudo systemctl stop nginx

なら、

> nginxを停止する

という意味。

### Q1の答え

**Q1. `sudo systemctl stop nginx` は何をする？**

→ **nginxを停止する。**

---

## 11. systemctl start

停止しているサービスを起動する。

    sudo systemctl start keyd

その後、

    systemctl is-active keyd

を実行。

結果：

    active

となった。

つまりkeydが起動した。

---

## 12. systemctl restart

サービスを再起動する。

    sudo systemctl restart keyd

確認：

    systemctl status keyd

結果：

    Active: active (running)

となった。

### restartの意味

    停止 → 起動

をまとめて行う。

設定ファイルを変更した後などに使用することが多い。

---

## 13. systemctl disable

サービスの自動起動を無効にする。

    sudo systemctl disable keyd

結果：

    Removed '/etc/systemd/system/multi-user.target.wants/keyd.service'.

確認：

    systemctl is-enabled keyd

結果：

    disabled

### 重要

`disable` は、

> 現在動作しているサービスを停止する

コマンドではない。

あくまで、

> 次回起動時などの自動起動設定を解除する

ためのコマンド。

実際に今回も、

    systemctl is-enabled keyd

は、

    disabled

になったが、

    systemctl is-active keyd

は、

    active

だった。

つまり、

    現在：起動中
    自動起動：無効

という状態になった。

---

## 14. systemctl enable

サービスの自動起動を有効にする。

    sudo systemctl enable keyd

確認：

    systemctl is-enabled keyd

結果：

    enabled

### Q2の答え

**Q2. `systemctl disable nginx` の意味は？**

→ **次回起動時などの自動起動設定を解除する。**

※現在動作中のnginxを停止するコマンドではない。

---

## 15. active + enabled

今回、最終的にkeydは、

    systemctl is-active keyd

    active

かつ、

    systemctl is-enabled keyd

    enabled

となった。

つまり、

    現在：起動していて稼働中
    自動起動：有効

という状態。

### Q3の答え

**Q3. `active` かつ `enabled` とは？**

→ **現在起動していて稼働中、かつ自動起動設定中。**

---

## 16. systemctl show

サービスの詳細なプロパティを確認できる。

今回：

    systemctl show keyd --property=ActiveState,SubState,UnitFileState

結果：

    ActiveState=active
    SubState=running
    UnitFileState=enabled

### それぞれの意味

    ActiveState=active

→ サービスが有効な状態。

    SubState=running

→ 実際にプロセスが実行中。

    UnitFileState=enabled

→ 自動起動設定が有効。

---

## 17. systemctl list-units

systemdが現在ロードしているユニットを確認：

    systemctl list-units --type=service

今回、

    65 loaded units listed.

と表示された。

### ポイント

`--type=service` を付けることで、サービスユニットに絞って表示できる。

---

## 18. active (running) と active (exited)

サービス一覧を見ると、

    loaded active running

だけでなく、

    loaded active exited

というものも存在した。

例えば、

    apparmor.service
    apport.service
    keyboard-setup.service
    ufw.service

など。

### active (running)

現在プロセスが動作し続けているタイプ。

例：

    chrony.service
    NetworkManager.service
    rsyslog.service

### active (exited)

起動時などに必要な処理を実行して、その処理が終了した状態。

これは必ずしも異常ではない。

つまり、

    active (exited) = 正常な場合もある

ということを学んだ。

---

## 19. systemctl --failed

systemdが管理しているユニットの中に、failed状態のものがあるか確認する。

    systemctl --failed

今回の結果：

    UNIT LOAD ACTIVE SUB DESCRIPTION

    0 loaded units listed.

### 意味

failed状態のユニットが0件。

つまり、

> systemdが管理しているサービス・ユニットに、現在failedになっているものはない

と判断できる。

### Q5の答え

**Q5. `systemctl --failed` は何を調べる？**

→ **systemdが管理しているサービスなどのユニットにfailed状態のものがあるか調べる。**

---

## 20. journalctl

systemdのログを確認するコマンド。

keydのログを確認した。

    journalctl -u keyd

### `-u`

特定のunitのログだけを表示する。

    journalctl -u keyd

なら、

> keyd.serviceに関するログを表示

する。

---

## 21. journalctl -n

最新のログだけ確認する。

    journalctl -u keyd -n 10

これは、

> keyd.serviceの最新10行のログを表示

する。

---

## 22. journalctl -b

現在のブート時のログに限定する。

    journalctl -u keyd -b

### `-b`

bootの略。

つまり、

> 今回のLinux起動以降に発生したkeydのログ

を確認できる。

過去の起動時ログと混ざらないため、現在の起動で何が起きているか確認するときに便利。

---

## 23. journalctl -f

ログをリアルタイムで追従する。

    journalctl -u keyd -f

### `-f`

followの意味。

新しいログが発生すると、その場で表示される。

例えば、

    journalctl -u nginx -f

としておけば、nginxのログをリアルタイムで監視できる。

### Q4の答え

**Q4. `journalctl -u keyd -f` の `-f` は？**

→ **ログをリアルタイムに追従する。**

---

## 24. keydのログから分かったこと

今回のログでは、

    CONFIG: parsing /etc/keyd/default.conf

が確認できた。

これはkeydが、

    /etc/keyd/default.conf

の設定ファイルを読み込んでいることを示している。

また、

    DEVICE: match 05ac:8105:2347b3ea /etc/keyd/default.conf
    (Apple Inc. Virtual USB Keyboard)

から、

> Appleの仮想USBキーボードに対してkeydの設定が適用されている

ことを確認できた。

---

## 25. 過去に発生していたkeydのエラー

過去のjournalには、

    ERROR: line 7: invalid key or action

というエラーも存在していた。

これは、

    /etc/keyd/default.conf

の7行目に、

> keydが認識できないキーまたはアクション

が記述されていたことを示している。

ただし、現在のログではこのエラーは発生していない。

現在は、

    CONFIG: parsing /etc/keyd/default.conf
    Starting keyd v2.5.0
    DEVICE: match ...

となっており、正常に起動できている。

---

## 26. systemctl list-dependencies

サービスの依存関係を確認できる。

    systemctl list-dependencies keyd

結果から、

    keyd.service
    ├─system.slice
    └─sysinit.target

などが確認できた。

さらに `sysinit.target` の下には、

    systemd-journald.service
    systemd-udevd.service
    systemd-resolved.service
    systemd-tmpfiles-setup.service
    ...

など、多くのsystemdユニットが存在していた。

### ポイント

Linuxのサービスは単独で存在しているわけではなく、

> 他のサービスやtarget、systemdの仕組みと依存関係を持ちながら動作している

ことが分かった。

---

## 27. systemdのサービス管理で重要なコマンド

| コマンド | 意味 |
|---|---|
| `systemctl status nginx` | nginxの状態確認 |
| `systemctl start nginx` | nginxを起動 |
| `systemctl stop nginx` | nginxを停止 |
| `systemctl restart nginx` | nginxを再起動 |
| `systemctl is-active nginx` | 現在起動中か確認 |
| `systemctl is-enabled nginx` | 自動起動設定を確認 |
| `systemctl enable nginx` | 自動起動を有効化 |
| `systemctl disable nginx` | 自動起動を無効化 |
| `systemctl --failed` | failed状態のユニットを確認 |
| `systemctl list-units --type=service` | サービス一覧 |
| `systemctl list-units --type=service --state=running` | 起動中のサービス一覧 |
| `systemctl list-dependencies nginx` | 依存関係を確認 |
| `systemctl show nginx` | 詳細なプロパティを確認 |
| `journalctl -u nginx` | nginxのログを確認 |
| `journalctl -u nginx -n 10` | 最新10行を確認 |
| `journalctl -u nginx -b` | 現在のブート以降のログ |
| `journalctl -u nginx -f` | ログをリアルタイム追従 |

---

## 28. 今日の重要ポイント

### systemctl stop

    sudo systemctl stop nginx

→ サービスを停止。

### systemctl start

    sudo systemctl start nginx

→ サービスを起動。

### systemctl restart

    sudo systemctl restart nginx

→ サービスを再起動。

### systemctl enable

    sudo systemctl enable nginx

→ 自動起動を有効化。

### systemctl disable

    sudo systemctl disable nginx

→ 自動起動を無効化。

### is-active

    systemctl is-active nginx

→ **今現在、サービスが動いているか。**

### is-enabled

    systemctl is-enabled nginx

→ **OS起動時に自動起動する設定か。**

### journalctl

    journalctl -u nginx

→ systemd管理サービスのログを確認。

### journalctl -f

    journalctl -u nginx -f

→ **ログをリアルタイムに追従。**

### systemctl --failed

    systemctl --failed

→ **failed状態のユニットを確認。**

---

## 29. 今日の理解確認

### Q1

    sudo systemctl stop nginx

何をする？

**答え：**

nginxを停止する。

---

### Q2

    sudo systemctl disable nginx

何をする？

**答え：**

次回起動時などの自動起動設定を解除する。

---

### Q3

    ActiveState=active
    SubState=running
    UnitFileState=enabled

この状態は？

**答え：**

起動していて稼働中、自動起動設定中。

---

### Q4

    journalctl -u nginx -f

`-f` の意味は？

**答え：**

ログをリアルタイムに追従する。

---

### Q5

    systemctl --failed

何を調べる？

**答え：**

systemdが管理しているサービスなどのユニットにfailed状態のものがあるか調べる。

---

## 30. インフラエンジニアとしての実践ポイント

サーバー障害が発生した場合、まずサービスの状態を確認する。

例えばnginxに問題がありそうなら、

    systemctl status nginx

↓

現在の状態を確認。

さらに、

    systemctl --failed

↓

failedしているユニットがないか確認。

さらに、

    journalctl -u nginx -n 50

↓

直近のログを確認。

必要であれば、

    journalctl -u nginx -f

↓

リアルタイムでログを監視する。

このように、

    状態確認
    ↓
    failed確認
    ↓
    ログ確認
    ↓
    原因調査
    ↓
    必要に応じてrestart

という流れで障害対応を行う。

---

## 31. 今日の学習まとめ

Day7では、Linuxのサービス管理の中心となるsystemdとsystemctlについて学習した。

特に重要なのは、

    systemctl status
    systemctl start
    systemctl stop
    systemctl restart
    systemctl enable
    systemctl disable
    systemctl is-active
    systemctl is-enabled
    systemctl --failed
    journalctl

の使い分け。

また、

    active

と

    enabled

は意味が異なることを理解した。

    active
    ↓
    現在サービスが動いているか

    enabled
    ↓
    OS起動時に自動起動する設定か

という違いがある。

さらに、`journalctl` を使ってsystemd管理サービスのログを確認し、

    journalctl -u サービス名 -f

によってログをリアルタイムで追跡できることも学習した。

今回のDay7で、Linuxサーバー上のサービスを

    起動
    停止
    再起動
    状態確認
    自動起動設定
    ログ確認
    障害確認
    依存関係確認

するための基本操作を一通り実践できた。

---

## 32. Day7で使用した主なコマンド

    ps -p 1 -o pid,comm,args

    systemctl status
    systemctl status chrony
    systemctl is-enabled chrony
    systemctl is-active chrony

    systemctl list-units --type=service --state=running
    systemctl list-units --type=service

    sudo systemctl stop keyd
    sudo systemctl start keyd
    sudo systemctl restart keyd

    sudo systemctl enable keyd
    sudo systemctl disable keyd

    systemctl is-active keyd
    systemctl is-enabled keyd

    systemctl show keyd --property=ActiveState,SubState,UnitFileState

    systemctl --failed

    systemctl list-dependencies keyd

    journalctl -u keyd
    journalctl -u keyd -n 10
    journalctl -u keyd -b
    journalctl -u keyd -f

---
