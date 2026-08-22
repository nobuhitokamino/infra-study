# Linux Day11 学習まとめ

## シェル（Shell）とbashの確認

Linuxでは、ユーザーの入力を受け取りコマンドを実行するプログラムをシェルという。

今回使用しているシェルはbash。

確認コマンド：

    echo $SHELL

実行結果：

    /bin/bash


現在実行中のシェル確認：

    ps -p $$

実行結果：

    PID TTY          TIME CMD
    4004 pts/0    00:00:00 bash


シェル名確認：

    echo $0

実行結果：

    /usr/bin/bash


---

# 環境変数とシェル変数

## シェル変数

現在のシェル内だけで利用できる変数。

例：

    TEST="hello linux"

確認：

    echo $TEST

結果：

    hello linux


ただし、子プロセスには渡らない。

確認：

    env | grep TEST

結果：

    （表示なし）


---

## 環境変数

exportすることで子プロセスへ引き継がれる変数。

設定：

    export TEST

確認：

    env | grep TEST

結果：

    TEST=hello linux


仕組み：

    親シェル
       |
       | export
       |
       ↓
    環境変数
       |
       ↓
    子プロセスへ継承


必要な情報だけを子プロセスへ渡し、不必要な情報は現在のシェル内に閉じ込めることで環境を整理できる。


---

# よく使う環境変数

ホームディレクトリ：

    echo $HOME

結果：

    /home/nobuhito


ユーザー名：

    echo $USER

結果：

    nobuhito


現在のディレクトリ：

    echo $PWD

結果：

    /home/nobuhito/infra-study/linux/linux-day11


PATH確認：

    echo $PATH


PATHとは、コマンドを検索する場所の一覧。

---

# コマンド検索

コマンドの場所確認：

    which ls

結果：

    /usr/bin/ls


コマンド種類確認：

    type ls

結果：

    ls is aliased to `ls --color=auto'


lsの実体確認：

    ls -l /usr/bin/ls


結果：

    /usr/bin/ls -> ../lib/cargo/bin/coreutils/ls


---

# .bashrcによる設定永続化

.bashrcはbash起動時に読み込まれる設定ファイル。

確認：

    ls -la ~

確認：

    tail ~/.bashrc


設定反映：

    source ~/.bashrc


新しいターミナルでも設定を有効にできる。

---

# シェルスクリプト

シェルスクリプトとは、複数のLinuxコマンドをまとめて自動実行する仕組み。

---

## 初めてのスクリプト

hello.sh作成：

    nano hello.sh


内容：

    #!/bin/bash

    echo "Hello Linux"
    echo "User: $USER"
    echo "Home: $HOME"


実行権限追加：

    chmod u+x hello.sh


実行：

    ./hello.sh


結果：

    Hello Linux
    User: nobuhito
    Home: /home/nobuhito


---

# シバン（shebang）

1行目：

    #!/bin/bash


意味：

    このスクリプトをbashで実行する


---

# 実行権限

確認：

    ls -l hello.sh


実行権限追加：

    chmod u+x hello.sh


意味：

    u → 所有者
    +x → 実行権限追加


---

# コマンドライン引数

スクリプトへ値を渡すことができる。

argument.sh：

    #!/bin/bash

    echo "1つ目の引数: $1"
    echo "2つ目の引数: $2"
    echo "引数の数: $#"


実行：

    ./argument.sh Linux Bash


結果：

    1つ目の引数: Linux
    2つ目の引数: Bash
    引数の数: 2


特殊変数：

    $0  → スクリプト名
    $1  → 1番目の引数
    $2  → 2番目の引数
    $#  → 引数の数


---

# 終了ステータス

Linuxコマンドは終了時に結果を返す。

確認：

    echo $?


成功：

    0


失敗：

    0以外


例：

    ls

    echo $?


結果：

    0


存在しないファイル：

    ls test123

    echo $?


結果：

    2


---

# if文（条件分岐）

条件によって処理を変える。

基本形：

    if 条件; then
        処理
    else
        処理
    fi


---

## ファイル存在確認

check_file.sh：

    #!/bin/bash

    if [ -f "$1" ]; then
        echo "$1 exists"
    else
        echo "$1 not found"
    fi


実行：

    ./check_file.sh hello.sh


結果：

    hello.sh exists


実行：

    ./check_file.sh test123


結果：

    test123 not found


---

# systemctlとシェルスクリプト

nginx状態確認：

    systemctl status nginx


状態：

    Active: active (running)


---

nginx監視スクリプト：

check_nginx.sh

    #!/bin/bash

    if systemctl is-active --quiet nginx; then
        echo "nginx is running"
    else
        echo "nginx is stopped"
    fi


実行結果：

    nginx is running


停止後：

    nginx is stopped


起動後：

    nginx is running


---

# for文（繰り返し）

決まった対象を順番に処理する。

loop.sh：

    #!/bin/bash

    for name in Linux Bash Shell
    do
        echo "Hello $name"
    done


実行：

    ./loop.sh


結果：

    Hello Linux
    Hello Bash
    Hello Shell


---

# while文（条件付き繰り返し）

条件がtrueの間処理を繰り返す。

while.sh：

    #!/bin/bash

    count=1

    while [ $count -le 5 ]
    do
        echo "count: $count"
        count=$((count + 1))
    done


実行結果：

    count: 1
    count: 2
    count: 3
    count: 4
    count: 5


計算：

    $((計算式))


例：

    count=$((count + 1))


---

# cron（定期実行）

cronとは、指定した時間に自動でコマンドやスクリプトを実行する仕組み。

確認：

    systemctl status cron


結果：

    Active: active (running)


---

cron設定確認：

    crontab -l


編集：

    crontab -e


---

# cron設定形式

    分 時 日 月 曜日 コマンド


例：

    * * * * * script.sh


意味：

    毎分実行


例：

    0 3 * * *


意味：

    毎日午前3時に実行


---

# cronによる自動ログ出力

cron_test.sh：

    #!/bin/bash

    echo "$(date) cron executed" >> /home/nobuhito/cron_test.log


手動実行：

    ./cron_test.sh


確認：

    cat /home/nobuhito/cron_test.log


結果：

    Sat Aug 22 03:14:22 PM JST 2026 cron executed


cron登録：

    * * * * * /home/nobuhito/infra-study/linux/linux-day11/cron_test.sh


確認：

    crontab -l


結果：

    * * * * * /home/nobuhito/infra-study/linux/linux-day11/cron_test.sh


自動実行後：

    Sat Aug 22 03:18:01 PM JST 2026 cron executed
    Sat Aug 22 03:19:01 PM JST 2026 cron executed
    Sat Aug 22 03:20:01 PM JST 2026 cron executed


---

# Day11で学んだこと

- bashの確認方法
- シェル変数と環境変数の違い
- exportによる環境変数化
- PATHの仕組み
- .bashrcによる設定永続化
- シェルスクリプト作成
- 実行権限
- シバン
- コマンドライン引数
- 終了ステータス
- if文
- for文
- while文
- systemctlとの連携
- cronによる定期実行
- Linuxでの簡単な運用自動化


# Day11の重要ポイント

Linux運用では、

    状態確認
        ↓
    スクリプト化
        ↓
    条件分岐
        ↓
    自動実行(cron)


という流れで、手作業を減らしていく。

これはインフラ運用、自動化、AWS運用でも基本となる考え方。
