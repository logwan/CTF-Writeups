# [Web] Plz Login

## 問題の概要
- **Difficulty:** Medium
- **Released:** Feb 9, 2026
- **Solved:** 2026-06-10
- **URL:** [https://alpacahack.com/challenges/plz-login](https://alpacahack.com/challenges/plz-login)

### 問題文
> ログインして！パスワードは教えないよ！

## 方針
指定された URL にアクセスすると、 `username` と `password` の入力欄がある。
`username` は、 `admin` か `Admin` ということが `app.py` の以下の部分で判明している。
```Python
if username[0] not in "aA" or username[1:] != "dmin" or password != "**REDACTED**":
        return render_template("login.html", error="You are not Admin"), 401
```
ただ、 `password` がわからない...。

## 解法
`password` は、どのファイルを見ても見つからなかったので、ファイルから探すのはあきらめた。 `app.py` からヒントを探したところ、以下の部分がヒントとなった。
```Python
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=3000, debug=True)
```
特に、 `debug=True` である。

これは、 `flask werkzeug デバッグ 情報漏洩` と検索すると出てくるが、Flaskのデバッグモード（ `debug=True` ）を本番環境で有効のままにすると、 Werkzeug の対話型デバッガーが外部に露出し、ソースコードの閲覧やリモートコード実行を許す壊滅的な情報漏洩につながるらしい。

このことより、何かしらエラーを起こせばよいのだとわかった。先ほど、 `username` が分かったコードを見てみると、 `username[0]` を調べている。では、 `username` が空だとどうなるのだろうか。

## 結果
`username` と `password` 欄に何も入力せずに `Login` ボタンを押すと `IndexError` が表示され、 `password` も見つかった。

## 感想
Flaskのデバッグモードについてよく知らなかったためよい機会となった。

#### 補足：Flaskのデバッグモード（Werkzeug）に関する詳細と考察
今回の脆弱性の根本的な原因は、本番環境で debug=True が有効になっていた点にある。
- **情報漏洩のメカニズム：** 
Flaskのデバッグモードでは、例外処理されていないエラーが発生すると「Werkzeug」の対話型デバッガが起動し、詳細なスタックトレースが画面に表示される。今回は `username` を空で送信することで意図的に `IndexError` （存在しない username[0] へのアクセス）を発生させ、デバッガ画面のローカル変数一覧から直前に格納されていた `password` を特定した。
- **本番環境における真の脅威:** 
このデバッガには、ブラウザ上で任意の Python コードを実行できる対話型コンソール機能が含まれている。つまり、単なる情報漏洩にとどまらず、 RCE　（リモートコード実行）によってサーバーの制御を完全に奪取される致命的なリスクを孕んでいる。
- **セキュアコーディングの観点からの対策:** 
    1. 本番環境では必ず環境変数などを利用して debug=False に設定する。
    2. 入力値が空文字である境界値のケースを想定し、配列のインデックスを参照する前に適切なチェック（例： `if not username:` など）を実装する。