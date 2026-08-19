# Linux学習 Day8 ネットワーク基礎

## 今日の学習内容

Day8では、Linuxサーバーのネットワーク基礎について学習した。

主に以下の内容を確認した。

- IPアドレスの確認
- ネットワークインターフェースの確認
- デフォルトゲートウェイの確認
- pingによる通信確認
- DNS名前解決
- HTTP通信確認
- 待ち受けポート確認
- IPアドレスとlocalhostの違い

---

# 1. IPアドレス確認

## 使用コマンド

    ip addr

    ip -br addr

    hostname -I

## 確認結果

Ubuntuのネットワーク情報を確認した。

    enp0s1
    IPv4: 192.168.64.3/24

今回のUbuntu環境では、

    192.168.64.3

がUbuntu自身に割り当てられたIPアドレスだった。

---

## ネットワークインターフェース

    enp0s1

は、Ubuntuが実際のネットワーク通信に利用しているインターフェース。

---

## lo（ループバック）

    lo

    127.0.0.1

はループバックインターフェース。

自分自身を表す特別なネットワーク。

---

# 2. IPアドレスとlocalhostの違い

## UbuntuのIPアドレス

例：

    192.168.64.3

これは環境によって変化する。

理由：

- DHCPによる自動割り当て
- 仮想環境のネットワーク設定変更
- 接続環境変更

などによって別のIPになる可能性がある。

---

## 127.0.0.1（localhost）

    127.0.0.1

は特別な予約アドレス。

意味：

    自分自身

を表す。

どのLinux環境でも基本的に同じ意味になる。

例：

    Ubuntu
    127.0.0.1

    AWS EC2
    127.0.0.1

どちらも、

    自分自身

を指す。

---

# 3. ルーティング確認

## 使用コマンド

    ip route

## 結果

    default via 192.168.64.1 dev enp0s1 proto dhcp src 192.168.64.3 metric 100

    192.168.64.0/24 dev enp0s1 proto kernel scope link src 192.168.64.3 metric 100

---

## デフォルトゲートウェイ

    192.168.64.1

がデフォルトゲートウェイだった。

役割：

    自分のネットワーク外へ通信するときの出口

通信イメージ：

    Ubuntu
    192.168.64.3

        ↓

    Gateway
    192.168.64.1

        ↓

    Internet

---

# 4. pingによる通信確認

## 自分自身への通信

    ping -c 4 127.0.0.1

結果：

    4 packets transmitted, 4 received, 0% packet loss

確認内容：

    Ubuntu自身のネットワーク機能が正常

---

## デフォルトゲートウェイへの通信

    ping -c 4 192.168.64.1

結果：

    4 packets transmitted, 4 received, 0% packet loss

確認内容：

    ローカルネットワーク内の通信が正常

---

## 外部IPへの通信

    ping -c 4 8.8.8.8

結果：

    4 packets transmitted, 4 received, 0% packet loss

確認内容：

    インターネットへのIP通信が正常

---

# 5. DNS（名前解決）

## pingで確認

    ping -c 4 google.com

結果：

    PING google.com (142.251.118.138)

    4 packets transmitted, 4 received, 0% packet loss

確認内容：

    google.comという名前をIPアドレスへ変換できた

---

## DNSとは

DNSとは、

    ドメイン名をIPアドレスへ変換する仕組み

のこと。

例：

    google.com

        ↓

    142.251.118.138

---

# 6. DNS設定確認

## 使用コマンド

    resolvectl status

確認結果：

    Current DNS Server: 192.168.64.1

    DNS Servers:
    192.168.64.1

今回の環境では、

    192.168.64.1

がDNSサーバーとして利用されていた。

---

## /etc/resolv.conf

    cat /etc/resolv.conf

確認結果：

    nameserver 127.0.0.53

127.0.0.53は実際のDNSサーバーではなく、

    systemd-resolved

が利用するローカルDNSスタブ。

流れ：

    アプリ

        ↓

    127.0.0.53

        ↓

    systemd-resolved

        ↓

    192.168.64.1

        ↓

    DNSサーバー

---

# 7. DNS名前解決確認

## resolvectl

    resolvectl query google.com

結果：

    google.com
    142.251.118.138

DNSによる名前解決が成功した。

---

## getent

    getent hosts google.com

結果：

    2404:6800:400a:1007::66 google.com

確認内容：

    Linuxの名前解決機能を利用して確認できる

---

# 8. HTTP通信確認

## HTML取得

    curl http://example.com

結果：

    Example Domain

HTMLレスポンスを取得できた。

---

## HTTPヘッダー確認

    curl -I http://example.com

結果：

    HTTP/1.1 200 OK

    Content-Type: text/html

    Server: cloudflare

確認内容：

    HTTP通信が正常に成功している

---

# HTTPステータスコード

代表的なもの：

    200
    成功

    301
    恒久的リダイレクト

    302
    一時的リダイレクト

    400
    リクエスト不正

    403
    アクセス禁止

    404
    Not Found

    500
    サーバー内部エラー

    502
    Bad Gateway

    503
    Service Unavailable

---

# 9. 待ち受けポート確認

## 使用コマンド

    ss -tuln

オプション：

    -t
    TCP

    -u
    UDP

    -l
    LISTEN状態

    -n
    名前解決せず数字表示

---

## 確認結果

DNS関連：

    127.0.0.53:53

    127.0.0.54:53

確認内容：

    systemd-resolvedがDNSポート53で待ち受けている

---

# ポートとは

IPアドレス：

    コンピューターの住所

ポート：

    サービスへの入口

例：

    192.168.64.3:22

の場合、

    IPアドレス
    192.168.64.3

    ポート
    22

を表す。

---

# 代表的なポート番号

    22
    SSH

    53
    DNS

    80
    HTTP

    443
    HTTPS

---

# 10. ネットワーク障害切り分け

例：

    ping 8.8.8.8

成功

    ↓

IP通信は正常


しかし、

    ping google.com

失敗

の場合、

疑うもの：

    DNS名前解決の問題

---

# Day8で学んだコマンドまとめ

    ip addr

    IPアドレス・ネットワークインターフェース確認


    ip route

    ルーティング・デフォルトゲートウェイ確認


    ping

    ICMPによる通信確認


    resolvectl

    DNS設定確認


    getent hosts

    名前解決確認


    curl

    HTTP通信確認


    ss -tuln

    待ち受けポート確認

---

# Day8まとめ

今回の学習で、

Linuxマシンがネットワーク通信を行う流れを理解した。

通信の流れ：

    IPアドレス確認

        ↓

    デフォルトゲートウェイ確認

        ↓

    IP通信確認

        ↓

    DNS名前解決

        ↓

    HTTP通信

        ↓

    ポート確認


インフラエンジニアとして重要な、

    IP
    Gateway
    DNS
    Port
    HTTP

の基本的な関係を理解できた。
