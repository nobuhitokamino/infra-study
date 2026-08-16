# Linux Day2 学習ログ

## ファイル操作

### echo

文字列を表示する

```bash
echo "Linux Day 2"
```
ファイル作成
```bash
echo "Linux Day 2" > test.txt
```
追記
```
echo "Command practice" >> test.txt
```
cat
ファイル内容表示
```
cat test.txt
```
結果
Linux Day 2
Command practice
grep
文字列検索
```
grep "Linux" test.txt
```
結果
Linux Day 2
パイプ利用
```
ls -l | grep txt
```
find
ファイル検索
```
find . -name "*.txt"
```
結果
./test.txt
./renamed.txt
Linuxディレクトリ構造
/
ルートディレクトリ
主なディレクトリ
```md
|場所|役割|
|-|-|
|/etc|	設定ファイル|
|/var/log|	ログ|
|/home|	ユーザー領域|
|/usr|	プログラム|
|/tmp|	一時ファイル|
```

ログ確認
syslog確認
```
tail /var/log/syslog
```
認証ログ
```
sudo tail /var/log/auth.log
```
sudo操作確認
```
sudo grep sudo /var/log/auth.log
```
今日の学び
Linuxでは設定ファイルは/etc、
ログは/var/logに保存される。
grepやfindを使うことで大量のファイルから
必要な情報を探せる。
インフラエンジニアにはログ確認能力が重要。

## 実行環境

- OS: Ubuntu
- User: nobuhito
- Directory: ~/infra-study/Linux
- Date: 2026-08-14
