---
title: SQLインジェクション
---

**SQLインジェクション攻撃について**

### **概要**

SQLインジェクション（SQL Injection）は、データベースと連携するアプリケーションがユーザーからの入力を適切に処理せずにSQLクエリとして直接使用してしまうことによって発生する脆弱性です。攻撃者は、この脆弱性を利用して、SQLクエリを改ざんし、データベースのデータを不正に操作（例えば、データの漏洩、削除、変更）することができます。

---

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP**

**脆弱なコード:**
```php
<?php
$username = $_GET['username'];
$password = $_GET['password'];

$query = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
$result = mysqli_query($conn, $query);

if (mysqli_num_rows($result) > 0) {
    echo "Login successful!";
} else {
    echo "Invalid username or password.";
}
?>
```

**攻撃手法:**
攻撃者は、`username`フィールドに`' OR '1'='1`を入力し、`password`フィールドに`' OR '1'='1`を入力することで、以下のようにクエリが改ざんされます。

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '' OR '1'='1';
```

このクエリは常に`true`を返し、攻撃者が認証を回避できるようになります。

#### **Python（Flask）**

**脆弱なコード:**
```python
from flask import request
import sqlite3

username = request.args.get('username')
password = request.args.get('password')

conn = sqlite3.connect('database.db')
cursor = conn.cursor()
query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
cursor.execute(query)

if cursor.fetchone():
    print("Login successful!")
else:
    print("Invalid username or password.")
```

**攻撃手法:**
同様に、`username`フィールドに`' OR '1'='1`、`password`フィールドに`' OR '1'='1`を入力することで、クエリが改ざんされ、認証が突破されます。

#### **Node.js（Express）**

**脆弱なコード:**
```javascript
const express = require('express');
const app = express();
const mysql = require('mysql');

const connection = mysql.createConnection({
    host: 'localhost',
    user: 'user',
    password: 'password',
    database: 'mydb'
});

app.get('/login', (req, res) => {
    const username = req.query.username;
    const password = req.query.password;

    const query = `SELECT * FROM users WHERE username = '${username}' AND password = '${password}'`;
    connection.query(query, (error, results) => {
        if (results.length > 0) {
            res.send("Login successful!");
        } else {
            res.send("Invalid username or password.");
        }
    });
});

app.listen(3000);
```

**攻撃手法:**
同様に、`username`フィールドに`' OR '1'='1`、`password`フィールドに`' OR '1'='1`を入力することで、クエリが改ざんされ、ログインが許可されてしまいます。

#### **Java（JDBC）**

**脆弱なコード:**
```java
import java.sql.*;

public class SQLInjectionExample {
    public static void main(String[] args) {
        String username = args[0];
        String password = args[1];

        String query = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'";

        try (Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mydb", "user", "password");
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(query)) {

            if (rs.next()) {
                System.out.println("Login successful!");
            } else {
                System.out.println("Invalid username or password.");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

**攻撃手法:**
Javaでも、`username`フィールドに`' OR '1'='1`、`password`フィールドに`' OR '1'='1`を入力することで、クエリが改ざんされ、認証を回避できます。

---

### **防止策**

1. **プリペアドステートメントとバインド変数の使用:**
   - クエリ内の動的部分をプレースホルダー（?）で置き換え、実際の値をバインドすることで、SQLインジェクションを防ぐことができます。
   - 例: PHPでは`mysqli_prepare`、Pythonでは`cursor.execute(query, (param1, param2))`、Node.jsでは`mysql2`モジュールの`prepared statements`、Javaでは`PreparedStatement`を使用します。

2. **入力のサニタイズとバリデーション:**
   - ユーザーからの入力をサニタイズし、SQL文に挿入されることを防ぎます。例えば、予期しない特殊文字やSQL文に関係するキーワードをエスケープします。
   - バリデーションも同時に行い、入力が期待された形式や範囲に収まっているかを確認します。

3. **データベースユーザーの権限を最小化:**
   - データベースのユーザー権限を最小限にすることで、万が一SQLインジェクションが成功しても、影響範囲を制限することができます。

4. **エラーメッセージの非表示化:**
   - エラーメッセージにデータベースやクエリ構造に関する情報が含まれていないようにし、攻撃者に手がかりを与えないようにします。

5. **Web Application Firewall（WAF）の導入:**
   - WAFは、SQLインジェクション攻撃を含む一般的な攻撃パターンを検出し、ブロックすることができます。

---

**まとめ:**
SQLインジェクションは、ユーザー入力を適切に処理しない場合に発生する脆弱性で、データベースの不正操作を可能にする危険な攻撃です。プリペアドステートメントの使用、入力サニタイズ、ユーザー権限の最小化などの防止策を適用することで、この脆弱性を効果的に防ぐことが可能です。各言語に共通して有効な対策であり、どのプラットフォームでもこれらのベストプラクティスを実装することが推奨されます。