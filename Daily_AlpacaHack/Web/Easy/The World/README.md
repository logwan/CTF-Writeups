# [Web] The World
## 問題の概要
- **Difficulty:** Easy
- **Released:** Feb 6, 2026
- **Solved:** 2026-05-26
- **URL:** https://alpacahack.com/daily/challenges/the-world

### 問題文
> 時よ止まれ！

## 方針
問題文やダウンロードできるファイルを確認すると、現在時刻の秒数を極限まで正確なタイミングで現在の秒数を送信する想定解も考えたが、ネットワークの遅延を考慮すると現実的ではないため、別の解法を探る必要があるのでは。

## 解法
ダウンロードできるフォルダの `server.sh` で以下の形で [Warmup] と [Impossible] を確認できる。
```sh
echo "[Warmup] current time (seconds)?"
read t; d1=$(( t-$(date +%s) ))
if (( -100 < d1 && d1 < 100 )); then
    echo "Well done."
else
    echo "Hm. diff: $d1"
    exit 1
fi

echo "[Impossible] current time (nanoseconds)?"
read t; d2=$(( t-$(date +%s%N) ))
if (( -100 < d2 && d2 < 100 )); then
    echo "The World! $FLAG"
else
    echo "Hm. diff: $d2"
    exit 1
fi
```
このプログラムの中でも `read t; d1=$(( t-$(date +%s) ))` に注目した。
入力した `t` は、値なら引き算が行われる。しかし、変数名として存在している文字列を入力すると自動的にその変数の中身を展開して計算しようとする。
これは、 `(())` や `$(( ))` （算術式評価）の性質である。


## 結果
入力に `FLAG` を入れると、以下のエラー文が出力される。
（答えの文字は伏字***にしています。）
```Bash
server.sh: line 6: Alpaca{***}: arithmetic syntax error: invalid arithmetic operator (error token is "{***}")

server.sh: line 7: d1: unbound variable Alpaca{***}
```
すぐには気づけなかったが、このエラー文に答えがあった。

Dockerfile を確認すると、 `socat` コマンドで `,stderr` オプションが付与されていた。これにより、本来サーバー内部にしか残らないはずのスクリプトのエラー出力（標準エラー出力）が、手元のネットワーク越しにそのまま返ってくる設定になっていた。

## 感想
なんとなくFLAG変数を入力したが、それで答えが出てくるとは思ってなかったので、解いた後に仕組みの理解をするのが大切なんだととても感じました。

## 今回初めて知ったこと
- 算術式評価の動き
