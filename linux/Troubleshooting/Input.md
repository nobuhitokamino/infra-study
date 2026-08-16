# Ubuntu 日本語入力（全角・半角切替）トラブルシューティング

## Ubuntu 日本語入力設定（Mozc）

## 発生した問題

Ubuntu Desktopをインストール後、ターミナルなどで日本語入力を試したところ、

- 英数字入力は可能
- 日本語入力（変換）ができない

状態だった。

確認すると、IBusの日本語入力エンジン（Mozc）がインストールされていなかった。

---

# Mozcの導入

日本語入力を可能にするためMozcをインストール。

確認コマンド：

```bash
ibus list-engine | grep mozc
```

結果：

```
mozc-off - Mozc:A_
mozc-jp - Mozc
mozc-on - Mozc:あ
```

Mozcの入力エンジンが利用可能になった。

---

# IBusの確認

現在使用している入力エンジン確認：

```bash
ibus engine
```

結果：

```
xkb:jp::jpn
```

Mozcへ変更：

```bash
ibus engine mozc-jp
```

確認：

```bash
ibus engine
```

結果：

```
mozc-jp
```

Mozcへの切り替え自体はコマンドでは成功した。

---

# 入力環境変数の確認

IBusが利用されているか確認。

```bash
echo $GTK_IM_MODULE
echo $QT_IM_MODULE
echo $XMODIFIERS
```

結果：

```
ibus
ibus
@im=ibus
```

IBusの設定は正常。

---

# 原因調査

## 原因1

Mac側のショートカット設定がUbuntu側の入力切替を邪魔していた。

Macでは、

```
Command + Space
```

がRaycast起動のショートカットに設定されていた。

そのためUbuntu側で、

```
Command + Space
```

を入力切替キーとして利用しようとしても、

Mac側で先にRaycastが反応していた。

対策：

Mac側のRaycastショートカットを解除。

---

## 原因2

Ubuntu側の入力ソース設定の問題。

最初は英数字入力しかできなかったため、日本語入力を追加する目的でMozcを導入した。

Mozc導入後、

```
ibus mozc-jp
```

で日本語入力はできるようになったが、

英数字入力へ戻す切り替えが正常に動作しなかった。

入力ソース確認：

```bash
gsettings get org.gnome.desktop.input-sources sources
```

一時的にMozcのみになっていた。

修正：

```bash
gsettings set org.gnome.desktop.input-sources sources "[('xkb', 'jp'), ('ibus', 'mozc-jp')]"
```

確認：

```bash
gsettings get org.gnome.desktop.input-sources sources
```

結果：

```
[('xkb', 'jp'), ('ibus', 'mozc-jp')]
```

英数字入力用の

```
xkb jp
```

と

日本語入力用の

```
ibus mozc-jp
```

を両方登録した。

---

# ショートカット設定

GNOME側で入力切替ショートカットを設定。

```bash
gsettings set org.gnome.desktop.wm.keybindings switch-input-source "['<Control>space']"
```

確認：

```bash
gsettings get org.gnome.desktop.wm.keybindings switch-input-source
```

結果：

```
['<Control>space']
```

ただし、この時点では正常に切り替わらなかった。

---

# コマンドによる確認

MozcのON/OFF切替は可能だった。

日本語入力：

```bash
ibus engine mozc-on
```

確認：

```bash
ibus engine
```

結果：

```
mozc-on
```

英数字入力：

```bash
ibus engine mozc-off
```

確認：

```bash
ibus engine
```

結果：

```
mozc-off
```

コマンドでは正常動作した。

---

# 最終的な解決

Ubuntuを再起動。

再起動後、

画面右上の入力メニューに

```
日本語 Mozc
英数字
```

の切替項目が表示されるようになった。

また、

```
Command + Space
```

で入力切替メニューが表示され、

日本語入力と英数字入力の切替が正常にできるようになった。

---

# 学んだこと

## Linuxでは入力設定は複数の要素で動いている

今回確認したもの：

- IBus
- Mozc
- GNOME入力ソース
- キーボード設定
- OS側ショートカット

単純にMozcを入れるだけではなく、

```
入力エンジン
↓
入力ソース
↓
ショートカット
↓
デスクトップ環境
```

全体の設定確認が必要。

---

## トラブルシューティングで使用したコマンド

### IBus確認

```bash
ibus engine
```

### 利用可能エンジン確認

```bash
ibus list-engine
```

### 入力ソース確認

```bash
gsettings get org.gnome.desktop.input-sources sources
```

### 現在の入力ソース確認

```bash
gsettings get org.gnome.desktop.input-sources current
```

### 環境変数確認

```bash
echo $GTK_IM_MODULE
echo $QT_IM_MODULE
echo $XMODIFIERS
```

---

# 学んだこと

LinuxのGUI設定は1つの設定だけではなく、

```
キーボード
 ↓
GNOME
 ↓
IBus
 ↓
Mozc
 ↓
アプリケーション
```

複数の仕組みが連携して動作している。

トラブル時は、

1. ハードウェア入力確認
2. OS側認識確認
3. 環境変数確認
4. サービス状態確認
5. 設定確認
6. 再起動による反映確認

という順番で原因を切り分けることが重要。

今回の日本語入力設定も、
Linuxサーバー障害調査と同じ考え方で解決できた。

---
# インフラエンジニアとしての学び

Linuxでは設定変更だけではなく、

- 現在の状態確認
- ログ確認
- サービス確認
- プロセス再起動
- 外部環境（ホストOS）の確認

まで含めて原因調査することが重要。

---
# インフラ学習としてのポイント

LinuxサーバーではGUI入力設定は少ないが、

- 設定ファイル確認
- ログ確認
- サービス状態確認
- 環境変数確認

という今回行った調査手順は、サーバートラブル対応でも共通する。

問題発生時は、

「何が動いているか」
「どこまで正常か」
「どの層で問題が起きているか」

を切り分けることが重要。
