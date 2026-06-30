# [Web] Alpaca Rangers

## 問題の概要
- **Difficulty:** Medium
- **Released:** Feb 25, 2026
- **Solved:** 2026-07-01
- **URL:** [https://alpacahack.com/challenges/alpaca-rangers](https://alpacahack.com/challenges/alpaca-rangers)

### 問題文
> 正義のヒーロー、アルパカレンジャー！

## 方針
指定された URL にアクセスすると、 `Red` `Blue` `Yellow` をクリックすることができる。
クリックすると、それぞれの色のヒーローを見ることができる。

## 解法
▼ すごく怪しい `index.php` の部分
```php
if ($targetPath !== '') {
    if (str_starts_with($targetPath, '/') || str_starts_with($targetPath, '\\') || str_contains($targetPath, '..')) {
        $errorMessage = 'Invalid path.';
    } else {
        $contents = @file_get_contents($targetPath);
        if ($contents === false) {
            $errorMessage = 'Not found.';
        } else {
            $finfo = new finfo(FILEINFO_MIME_TYPE);
            $mimeType = $finfo->buffer($contents) ?: 'application/octet-stream';

            $dataUri = 'data:' . $mimeType . ';base64,' . base64_encode($contents);
        }

    }
}
```
この中でも、以下の条件に注目した。
```php
if (str_starts_with($targetPath, '/') ||
 str_starts_with($targetPath, '\\') ||
 str_contains($targetPath, '..'))
```
意味としては、

- `/` で始まると✖
- `\` で始まると✖
- `..` を含むと✖

また、 `index.php` の以下の部分にも注目した。
```php
$targetPath = $_GET['img'] ?? '';
```
この文より、 `http://~~/?img=` の形でしか `flag.txt` にたどり着けないことがわかる。

しかし、`http://~~/?img=flag.txt` で行けるわけがないので...。

ここで、 `file_get_contents` について調査した。

▼ 参考にしたサイト
- https://www.php.net/manual/ja/function.file-get-contents.php
- https://www.php.net/manual/ja/wrappers.php

これらのサイトによると、 `file_get_contents` がストリームラッパー経由で `/flag.txt` を読みに行くことができるそう。

そして、読み込んだ内容はそのまま表示されるのではなく、`base64` に変換されて `data:` URL の形で `<img>` タグに埋め込まれる。

`base64` は、バイナリや文字列を 64 種類の記号だけで表現するエンコード方式で、Web ページ中に安全に埋め込むときによく使われる。
今回の問題では、フラグ本体はページ上に直接見えず、HTML ソース内の `data:text/plain;base64,...` の `...` 部分に隠れていた。


## 結果
https://www.php.net/manual/ja/wrappers.php.php

上記のURLより、 `php://filter` が使えそう。

```
http://~~/?img=php://filter/resource=/flag.txt
```
これから得られた文字列に `CyberChef` で `From Base64` を適用するとフラッグが入手できた。


## 感想
プロトコルは知ってるけど、ラッパー？

## 補足
### 1. 今回の問題の種類
**Path Traversal（パストラバーサル）**
    
本来アクセスできないはずのファイルに、パスを工夫することでアクセスする攻撃手法。
### 2. PHPストリームラッパーってなに？
感想で触れた「ラッパー」について。

PHPにおけるストリームラッパーは、ファイルシステム以外の様々なデータソース（HTTPリソース、FTP、標準入出力など）を、通常のファイル読み書き関数（`file_get_contents` や `fopen` など）で同じように扱えるようにするための機能。

`php://` はPHPが独自に提供するラッパー群で、その中の `php://filter` は、データを読み込む際にbase64エンコードなどの「フィルタ（変換）」を挟むことができる強力な機能。