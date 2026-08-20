# Linux学習 Day9 — ネットワーク通信の実践

## 今日のテーマ

Day8で学んだネットワーク基礎を、実際のLinux環境で確認した。

今回のDay9では、以下を中心に学習した。

- `ss` によるポート・TCP接続の確認
- `LISTEN` / `ESTABLISHED` / `TIME-WAIT`
- nginxのインストールとsystemdによる管理
- `127.0.0.1` と `0.0.0.0` の違い
- TCP 80番ポートとHTTP
- DNSによる名前解決
- IPv4 / IPv6
- HTTPS、TLS、443番ポート
- `curl -v` による通信の確認
- UFWとファイアウォール
- AWS Security Groupとの関連

---

## 1. `ss` コマンドで待ち受けポートを確認

`ss` はLinuxのネットワークソケット情報を確認するコマンド。

主なオプション：

- `-t`：TCP
- `-u`：UDP
- `-l`：LISTEN状態
- `-n`：名前解決せずポート番号をそのまま表示

実行結果では、DNSやlocalhost関連のポートなどが確認できた。

    ss -tuln

    Netid     State      Recv-Q     Send-Q      Local Address:Port
    ...
    tcp       LISTEN     0          4096       127.0.0.53:53
    tcp       LISTEN     0          511        127.0.0.1:33255
    tcp       LISTEN     0          4096       127.0.0.1:631
    ...

---

## 2. `ss -tulnp` でプロセスまで確認

`-p` を追加すると、そのポートを使用しているプロセスを確認できる。

    ss -tulnp

例えば、

    tcp    LISTEN    0    511    127.0.0.1:33255    0.0.0.0:*    users:(("code",pid=8285,fd=53))

これは、

    PID 8285
        ↓
    VS Code
        ↓
    Ubuntu自身の127.0.0.1:33255
        ↓
    TCP通信を待ち受け

という意味。

### `127.0.0.1` と `0.0.0.0`

    127.0.0.1
        ↓
    localhost
        ↓
    自分自身

    0.0.0.0
        ↓
    IPv4の全ローカルインターフェース

`0.0.0.0:80` だからといって、必ず外部からアクセスできるわけではない。

ファイアウォールやAWSのSecurity Groupなど、別の通信制御も存在する。

---

## 3. nginxをインストール

最初にnginxの状態を確認した。

    systemctl status nginx

    Unit nginx.service could not be found.

nginxがインストールされていなかったため、インストールした。

    sudo apt update

    sudo apt install nginx

インストール後、systemdのサービスとしてnginxが登録された。

    systemctl status nginx

    ● nginx.service - A high performance web server and a reverse proxy server
         Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
         Active: active (running)
         ...
         Main PID: 5277 (nginx)

### 確認できたこと

    Loaded: loaded
        ↓
    nginx.serviceがsystemdに登録されている

    enabled
        ↓
    システム起動時に自動起動する設定

    Active: active (running)
        ↓
    現在nginxが起動している

---

## 4. nginxが80番ポートでLISTENしていることを確認

    ss -tuln | grep ':80'

    tcp   LISTEN   0   511   0.0.0.0:80   0.0.0.0:*
    tcp   LISTEN   0   511   [::]:80      [::]:*

nginxがTCPの80番ポートで待ち受けていることを確認した。

さらにプロセスも確認した。

    sudo ss -tulnp | grep ':80'

    tcp   LISTEN   0   511   0.0.0.0:80   0.0.0.0:*   users:(("nginx",pid=5282,...),("nginx",pid=5281,...),("nginx",pid=5280,...),("nginx",pid=5279,...),("nginx",pid=5277,...))
    tcp   LISTEN   0   511   [::]:80      [::]:*       users:(("nginx",pid=5282,...),("nginx",pid=5281,...),("nginx",pid=5280,...),("nginx",pid=5279,...),("nginx",pid=5277,...))

### ポイント

    0.0.0.0:80
        ↓
    IPv4の全ローカルインターフェースで
    TCP 80番ポートを待ち受け

    [::]:80
        ↓
    IPv6側でも80番ポートを待ち受け

---

## 5. curlでnginxへアクセス

localhostにアクセスした。

    curl http://127.0.0.1

    <!DOCTYPE html>
    <html>
    ...
    <h1>Welcome to nginx!</h1>
    ...
    </html>

nginxのデフォルトページが返ってきた。

つまり、

    curl
      ↓
    127.0.0.1:80
      ↓
    nginx
      ↓
    HTTPレスポンス
      ↓
    HTML

という通信が成功した。

---

## 6. `curl -I` でHTTPヘッダーを確認

    curl -I http://127.0.0.1

    HTTP/1.1 200 OK
    Server: nginx/1.28.3 (Ubuntu)
    Date: Thu, 20 Aug 2026 12:58:10 GMT
    Content-Type: text/html
    Content-Length: 615
    Last-Modified: Thu, 20 Aug 2026 12:33:21 GMT
    Connection: keep-alive
    ETag: "6a86f411-267"
    Accept-Ranges: bytes

### `200 OK`

HTTPステータスコード。

    200 OK
        ↓
    HTTPリクエストが正常に処理された

### `Server`

    Server: nginx/1.28.3 (Ubuntu)

レスポンスを返したWebサーバーがnginxであることが分かる。

---

## 7. `curl -v` でTCPとHTTPの詳細を確認

    curl -v http://127.0.0.1

重要な部分：

    * Trying 127.0.0.1:80...
    * Established connection to 127.0.0.1 (127.0.0.1 port 80) from 127.0.0.1 port 45022

    > GET / HTTP/1.1
    > Host: 127.0.0.1
    > User-Agent: curl/8.18.0
    > Accept: */*

    < HTTP/1.1 200 OK
    < Server: nginx/1.28.3 (Ubuntu)

ここから、

    クライアント側
    127.0.0.1:45022
          ↓
    サーバー側
    127.0.0.1:80

というTCP接続が確立したことが分かる。

45022はクライアント側に一時的に割り当てられたポート。

---

## 8. TCPの状態を確認

    ss -tan

主な状態：

    LISTEN
        ↓
    新しいTCP接続を待ち受けている

    ESTAB / ESTABLISHED
        ↓
    TCP接続が確立している

    TIME-WAIT
        ↓
    TCP接続終了後、一定時間その接続情報を保持している状態

実際に、

    watch -n 1 'ss -tan | grep ":80"'

を使ってリアルタイムに確認した。

    LISTEN              0    511    0.0.0.0:80       0.0.0.0:*
    TIME-WAIT           0    0      127.0.0.1:36776  127.0.0.1:80
    TIME-WAIT           0    0      127.0.0.1:36786  127.0.0.1:80
    LISTEN              0    511    [::]:80          [::]:*

curlによるHTTP通信が終了したあと、TCP接続がTIME-WAITになることを実際に確認できた。

---

## 9. UFWの状態を確認

    sudo ufw status

    Status: inactive

UFWは現在無効になっていた。

ここで重要なのは、

    UFW inactive
        ↓
    UFWによる通信制御は現在行われていない

ということ。

ただし、

    UFW inactive
        ↓
    Linux上のあらゆるファイアウォール機能が存在しない

という意味ではない。

---

## 10. Ubuntuの実IPアドレスからnginxへアクセス

UbuntuのIPアドレスは、

    192.168.64.3

だった。

このIPアドレスを使ってnginxへアクセスした。

    curl http://192.168.64.3

    <!DOCTYPE html>
    <html>
    ...
    <h1>Welcome to nginx!</h1>
    ...
    </html>

これにより、

    127.0.0.1:80
        ↓
    localhostからアクセス

だけではなく、

    192.168.64.3:80
        ↓
    Ubuntuのネットワークインターフェース経由でアクセス

できることを確認した。

これはnginxが、

    0.0.0.0:80

で待ち受けていることとも関係している。

---

## 11. `0.0.0.0` の重要な意味

例えば、

    127.0.0.1:8080

なら、

    localhost
        ↓
    自分自身からの接続

に限定される。

一方、

    0.0.0.0:8080

なら、

    IPv4の全ローカルインターフェース
        ↓
    8080番ポートで待ち受け

となる。

ただし、

    0.0.0.0:8080
        ↓
    必ず外部からアクセス可能

ではない。

外部からアクセスできるかどうかは、

    サービスのLISTEN
        +
    ファイアウォール
        +
    ネットワーク設定
        +
    AWSの場合はSecurity Groupなど

によって決まる。

---

## 12. DNSによる名前解決

`example.com` の名前解決を確認した。

    getent hosts example.com

    2606:4700:10::6814:179a example.com
    2606:4700:10::ac42:93f3 example.com

IPv6アドレスが返ってきた。

IPv4も確認した。

    getent ahostsv4 example.com

    104.20.23.154   STREAM example.com
    104.20.23.154   DGRAM
    104.20.23.154   RAW
    172.66.147.243  STREAM
    172.66.147.243  DGRAM
    172.66.147.243  RAW

この結果から、

    example.com
        ↓
    104.20.23.154
    172.66.147.243

のように、1つのドメイン名に複数のIPv4アドレスが対応していることを確認した。

### 名前解決の基本

    ドメイン名
        ↓
       DNS
        ↓
    IPアドレス

---

## 13. `curl -v http://example.com` で実際のインターネット通信を確認

    curl -v http://example.com

重要な部分：

    * Host example.com:80 was resolved.
    * IPv6: 2606:4700:10::ac42:93f3, 2606:4700:10::6814:179a
    * IPv4: 172.66.147.243, 104.20.23.154

IPv6を試したところ、

    * Trying [2606:4700:10::ac42:93f3]:80...
    * Immediate connect fail for 2606:4700:10::ac42:93f3: Network is unreachable

となった。

これはDNS失敗ではない。

DNSによってIPv6アドレスは正常に取得できている。

現在のUbuntu環境にはIPv6で通信するためのネットワーク経路がないため、IPv6接続に失敗した。

その後IPv4で、

    * Trying 172.66.147.243:80...
    * Established connection to example.com (172.66.147.243 port 80) from 192.168.64.3 port 32990

となり、IPv4で接続成功した。

通信の流れは、

    example.com
        ↓
       DNS
        ↓
    IPv6 / IPv4アドレス
        ↓
    IPv6接続失敗
        ↓
    IPv4へ
        ↓
    172.66.147.243:80
        ↓
    TCP接続
        ↓
    HTTP
        ↓
    GET /
        ↓
    HTTP/1.1 200 OK

となった。

---

## 14. HTTP通信の流れ

今回の実験から、HTTP通信は次のように考えられる。

    ドメイン名
        ↓
       DNS
        ↓
    IPアドレス
        ↓
    TCP :80
        ↓
      HTTP
        ↓
    Webサーバー
        ↓
    HTTPレスポンス

HTTPでは基本的にTLSによる暗号化は行われない。

---

## 15. HTTPS通信を確認

    curl -v https://example.com

まず、

    * Host example.com:443 was resolved.

となった。

HTTPSでは標準的にTCP 443番ポートを使用する。

IPv6は今回も、

    * Trying [2606:4700:10::6814:179a]:443...
    * Immediate connect fail for 2606:4700:10::6814:179a: Network is unreachable

となり、IPv4へ切り替わった。

その後、

    * Trying 104.20.23.154:443...

で接続成功。

---

## 16. HTTPSではTLSハンドシェイクが行われる

HTTPS通信では、

    TCP :443
        ↓
    TLSハンドシェイク
        ↓
    暗号化通信の準備
        ↓
    HTTP通信

という流れになる。

実際のログでも、

    * TLSv1.3 (OUT), TLS handshake, Client hello (1):
    * TLSv1.3 (IN), TLS handshake, Server hello (2):
    * TLSv1.3 (IN), TLS handshake, Certificate (11):
    ...
    * SSL connection using TLSv1.3

と確認できた。

今回はTLS 1.3が使用された。

---

## 17. サーバー証明書の確認

HTTPS通信ではサーバー証明書も確認された。

    * Server certificate:
    * subject: CN=example.com

さらに、

    * subjectAltName: "example.com" matches cert's "example.com"
    * SSL certificate verified via OpenSSL.

となっていた。

つまり、curlがサーバー証明書を検証し、

    example.com
        ↓
    証明書の名前と一致
        ↓
    証明書の検証成功

となった。

---

## 18. HTTPSではHTTP/2も確認できた

今回、

    * using HTTP/2

となっていた。

リクエストも、

    > GET / HTTP/2
    > Host: example.com

となっていた。

レスポンスは、

    < HTTP/2 200

だった。

つまり今回のHTTPS通信は、

    DNS
      ↓
    IPv4
      ↓
    TCP :443
      ↓
    TLS 1.3
      ↓
    HTTP/2
      ↓
    GET /
      ↓
    HTTP/2 200

という流れだった。

---

## 19. HTTPとHTTPSの違い

### HTTP

    DNS
      ↓
    IPアドレス
      ↓
    TCP :80
      ↓
    HTTP

### HTTPS

    DNS
      ↓
    IPアドレス
      ↓
    TCP :443
      ↓
    TLS
      ↓
    HTTP

HTTPSではTLSによって通信を保護する。

---

## 20. AWSとの関連

今回学んだ内容は、AWSのインフラ構築にも直接つながる。

例えば、

    ブラウザ
        ↓
    HTTPS :443
        ↓
    ALB
        ↓
    HTTP :80
        ↓
    EC2 / nginx
        ↓
    Spring Boot :8080

という構成が考えられる。

このとき、

    443
        ↓
    HTTPS

    TLS
        ↓
    証明書による安全な通信

    80
        ↓
    HTTP

    8080
        ↓
    Spring Bootなどのアプリケーション

という役割分担になる。

さらに、

    Security Group
        ↓
    許可するポートを制御

という仕組みもある。

そのため、サーバーに接続できない場合は、

    ① サービスが起動しているか
        ↓
    ② ポートでLISTENしているか
        ↓
    ③ ファイアウォールで遮断されていないか
        ↓
    ④ ネットワーク経路に問題がないか
        ↓
    ⑤ AWSならSecurity Groupなどを確認

という切り分けができる。

---

# Day9で覚えたコマンド

    ss -tuln
    ss -tulnp
    ss -tan
    systemctl status nginx
    sudo apt update
    sudo apt install nginx
    sudo ss -tulnp | grep ':80'
    curl http://127.0.0.1
    curl -I http://127.0.0.1
    curl -v http://127.0.0.1
    curl http://192.168.64.3
    sudo ufw status
    getent hosts example.com
    getent ahostsv4 example.com
    curl -v http://example.com
    curl -v https://example.com
    watch -n 1 'ss -tan | grep ":80"'

---

# Day9の重要用語

| 用語 | 意味 |
|---|---|
| LISTEN | TCP接続を待ち受けている状態 |
| ESTABLISHED | TCP接続が確立している状態 |
| TIME-WAIT | TCP接続終了後、一定時間接続情報を保持する状態 |
| localhost | 自分自身のホスト |
| 127.0.0.1 | localhostを表す代表的なIPv4アドレス |
| 0.0.0.0 | IPv4の全ローカルインターフェース |
| TCP | 通信を確立してデータを送受信するプロトコル |
| DNS | ドメイン名をIPアドレスなどに名前解決する仕組み |
| HTTP | Web通信に使用されるプロトコル |
| HTTPS | TLSで保護されたHTTP通信 |
| TLS | 通信を暗号化・認証するためのプロトコル |
| nginx | Webサーバー・リバースプロキシ |
| UFW | Ubuntuで利用できるファイアウォール管理ツール |
| Port | 通信先のサービスを識別する番号 |

---

# Day9の最重要ポイント

今回の学習で特に重要なのは、**「サービスが動いている」ことと「ネットワークからアクセスできる」ことは別の問題**だということ。

例えばnginxの場合、

    systemctl status nginx
        ↓
    nginxが起動している

だけでは不十分。

さらに、

    ss -tuln
        ↓
    TCP :80でLISTENしている

ことを確認する。

さらに外部からの通信なら、

    ファイアウォール
        ↓
    ネットワーク経路
        ↓
    AWS Security Group
        ↓
    などを確認する。

また、Webアクセスでは、

    ドメイン名
        ↓
    DNS
        ↓
    IPアドレス
        ↓
    TCP
        ↓
    Port
        ↓
    TLS（HTTPSの場合）
        ↓
    HTTP
        ↓
    Webサーバー
        ↓
    レスポンス

という一連の流れを意識する。

---

# Day9まとめ

Day8ではネットワークの基本概念を学習し、Day9ではそれを実際のLinux環境で確認した。

nginxを起動して80番ポートでLISTENしていることを確認し、`curl`によって実際のHTTP通信を行った。

さらに`ss`を使用して、TCPの`LISTEN`、`ESTABLISHED`、`TIME-WAIT`といった状態を確認した。

その後、`getent`によってDNS名前解決を確認し、`curl -v`を使用して、

    DNS
      ↓
    IPアドレス
      ↓
    TCP
      ↓
    HTTP

という通信の流れを確認した。

HTTPSでは、

    DNS
      ↓
    IPアドレス
      ↓
    TCP :443
      ↓
    TLS 1.3
      ↓
    HTTP/2

という流れになり、TLSハンドシェイクやサーバー証明書の検証も実際のログから確認できた。

今回のDay9を通して、**Linuxのサービス・プロセス・ポート・TCP・DNS・HTTP/HTTPSが、それぞれ独立した知識ではなく、実際のネットワーク通信の中で繋がっている**ことを理解できた。
