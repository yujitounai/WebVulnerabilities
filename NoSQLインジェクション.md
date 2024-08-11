---
title: NoSQLインジェクション
---


### NoSQLインジェクションについて

#### 概要

NoSQLインジェクションは、NoSQLデータベースに対して行われるインジェクション攻撃で、SQLインジェクションに似た手法です。攻撃者は、アプリケーションがNoSQLクエリを動的に生成する際に、ユーザー入力を利用してクエリを不正に操作し、予期しないデータの取得や不正な操作を行うことができます。特に、MongoDBのようなドキュメント指向のデータベースで多く見られますが、他のNoSQLデータベースにも同様の脆弱性が存在します。

#### 脆弱性を持つコードの例と攻撃手法

以下では、MongoDBを使用した典型的なNoSQLインジェクション攻撃の例を、PHP、Python、Node.js、Javaで説明します。

##### PHP

**脆弱性を持つコードの例:**

```php
<?php
// MongoDBに接続
$client = new MongoDB\Client("mongodb://localhost:27017");
$collection = $client->mydatabase->users;

$username = $_GET['username'];
$password = $_GET['password'];

// クエリの構築
$query = array("username" => $username, "password" => $password);
$user = $collection->findOne($query);

if ($user) {
    echo "Login successful!";
} else {
    echo "Login failed!";
}
```

**攻撃手法:**

攻撃者は、`username`または`password`パラメータに特殊な入力を挿入します。

例:
```
?username=admin&password[$ne]=1
```

この場合、クエリは以下のように構築されます:
```php
$query = array("username" => "admin", "password" => array('$ne' => 1));
```

このクエリは、`username`が`admin`であり、`password`が`1`以外のすべてのドキュメントを返すため、実質的にどのパスワードでもログインが成功します。

##### Python (FlaskとPyMongo)

**脆弱性を持つコードの例:**

```python
from flask import Flask, request
from pymongo import MongoClient

app = Flask(__name__)
client = MongoClient('localhost', 27017)
db = client.mydatabase

@app.route('/login')
def login():
    username = request.args.get('username')
    password = request.args.get('password')

    query = {"username": username, "password": password}
    user = db.users.find_one(query)

    if user:
        return "Login successful!"
    else:
        return "Login failed!"
```

**攻撃手法:**

攻撃者がURLに以下のように入力すると、
```
/login?username=admin&password[$ne]=1
```
`password`が`1`以外であることを条件に、ユーザーが見つかり、ログインが成功します。

##### Node.js (ExpressとMongoDB)

**脆弱性を持つコードの例:**

```javascript
const express = require('express');
const MongoClient = require('mongodb').MongoClient;

const app = express();
const url = 'mongodb://localhost:27017';
const dbName = 'mydatabase';

MongoClient.connect(url, (err, client) => {
    const db = client.db(dbName);
    const users = db.collection('users');

    app.get('/login', (req, res) => {
        const username = req.query.username;
        const password = req.query.password;

        users.findOne({ username: username, password: password }, (err, user) => {
            if (user) {
                res.send("Login successful!");
            } else {
                res.send("Login failed!");
            }
        });
    });

    app.listen(3000);
});
```

**攻撃手法:**

攻撃者が以下のようにリクエストを送ると、
```
/login?username=admin&password[$ne]=1
```
同様に、任意のパスワードでログインが成功します。

##### Java (MongoDB Java Driver)

**脆弱性を持つコードの例:**

```java
import com.mongodb.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class LoginServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        MongoClient mongoClient = new MongoClient("localhost", 27017);
        MongoDatabase database = mongoClient.getDatabase("mydatabase");
        MongoCollection<Document> collection = database.getCollection("users");

        String username = req.getParameter("username");
        String password = req.getParameter("password");

        Document query = new Document("username", username).append("password", password);
        Document user = collection.find(query).first();

        if (user != null) {
            resp.getWriter().write("Login successful!");
        } else {
            resp.getWriter().write("Login failed!");
        }

        mongoClient.close();
    }
}
```

**攻撃手法:**

同様に、攻撃者が以下のリクエストを送ると、
```
/login?username=admin&password[$ne]=1
```
任意のパスワードでログインが成功します。

#### 防止策

1. **ユーザー入力の検証・サニタイズ**:
   - ユーザーからの入力データを常に検証し、不正な構文や操作を含むデータを拒否します。特に、クエリパラメータとして渡される配列やオブジェクトを慎重に検証します。

2. **ORM/ODMの使用**:
   - オブジェクトリレーショナルマッピング（ORM）やオブジェクトドキュメントマッピング（ODM）を利用することで、クエリを直接作成することなく、安全にデータ操作を行うことができます。

3. **パラメータ化クエリ**:
   - SQLのプリペアドステートメントと同様に、NoSQLでも可能な限りパラメータ化されたクエリを使用し、ユーザー入力をクエリに直接埋め込まないようにします。

4. **制限付きの操作**:
   - データベース操作に関しては、ユーザー入力を利用する際に、明確に定義されたフィールド名や型に基づく操作のみを許可します。

5. **ライブラリやフレームワークの利用**:
   - NoSQLインジェクションのリスクを軽減するために、セキュリティ機能を提供するライブラリやフレームワークを使用します。

NoSQLインジェクションは、NoSQLデータベースに対して不正な入力を利用してクエリを操作する攻撃手法です。`$ne`（not equal）以外にも、さまざまな方法でクエリを操作して攻撃を行うことができます。以下では、他の攻撃手法をいくつか紹介します。

### 1. **`$or`を利用した攻撃**

**概要:**
`$or`演算子は、複数の条件のいずれかが真であれば、クエリが一致するレコードを返します。攻撃者はこれを利用して、任意の条件を満たすレコードを取得することができます。

**脆弱性を持つコードの例:**

- **Node.js (ExpressとMongoDB)**

```javascript
const express = require('express');
const MongoClient = require('mongodb').MongoClient;

const app = express();
const url = 'mongodb://localhost:27017';
const dbName = 'mydatabase';

MongoClient.connect(url, (err, client) => {
    const db = client.db(dbName);
    const users = db.collection('users');

    app.get('/login', (req, res) => {
        const username = req.query.username;
        const password = req.query.password;

        users.findOne({ username: username, password: password }, (err, user) => {
            if (user) {
                res.send("Login successful!");
            } else {
                res.send("Login failed!");
            }
        });
    });

    app.listen(3000);
});
```

**攻撃手法:**

攻撃者は、`username`または`password`パラメータに`$or`を挿入して、常に`true`となる条件を作り出します。

例:
```
/login?username=admin&password[$or]=[{password: 'anything'}, {1: 1}]
```

これにより、クエリは次のようになります:
```javascript
{ username: 'admin', $or: [{ password: 'anything' }, { 1: 1 }] }
```

このクエリは、`username`が`admin`であれば、`password`が何であってもログインが成功します。

### 2. **`$regex`を利用した攻撃**

**概要:**
`$regex`演算子は、正規表現を使った検索を可能にします。攻撃者はこれを利用して、特定の条件を無視して任意のデータを取得することができます。

**脆弱性を持つコードの例:**

- **Python (FlaskとPyMongo)**

```python
from flask import Flask, request
from pymongo import MongoClient

app = Flask(__name__)
client = MongoClient('localhost', 27017)
db = client.mydatabase

@app.route('/login')
def login():
    username = request.args.get('username')
    password = request.args.get('password')

    query = {"username": username, "password": password}
    user = db.users.find_one(query)

    if user:
        return "Login successful!"
    else:
        return "Login failed!"
```

**攻撃手法:**

攻撃者は、`password`パラメータに`$regex`を挿入して、任意のパスワードでログインを成功させることができます。

例:
```
/login?username=admin&password[$regex]=.*
```

これにより、クエリは次のようになります:
```python
{"username": "admin", "password": {"$regex": ".*"}}
```

このクエリは、`username`が`admin`であれば、どのパスワードでもログインが成功します。

### 3. **`$gt`, `$lt`を利用した範囲攻撃**

**概要:**
`$gt`（greater than）や`$lt`（less than）演算子を使用して、特定の範囲の条件を操作することができます。これを利用して、攻撃者は条件を回避し、データを取得できます。

**脆弱性を持つコードの例:**

- **Java (MongoDB Java Driver)**

```java
import com.mongodb.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class LoginServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        MongoClient mongoClient = new MongoClient("localhost", 27017);
        MongoDatabase database = mongoClient.getDatabase("mydatabase");
        MongoCollection<Document> collection = database.getCollection("users");

        String username = req.getParameter("username");
        String password = req.getParameter("password");

        Document query = new Document("username", username).append("password", password);
        Document user = collection.find(query).first();

        if (user != null) {
            resp.getWriter().write("Login successful!");
        } else {
            resp.getWriter().write("Login failed!");
        }

        mongoClient.close();
    }
}
```

**攻撃手法:**

攻撃者は、`password`に`$gt`や`$lt`を利用して範囲外の条件を作り出します。

例:
```
/login?username=admin&password[$gt]=0
```

これにより、クエリは次のようになります:
```java
{"username": "admin", "password": {"$gt": 0}}
```

このクエリは、`username`が`admin`であれば、`password`が数値の0より大きいすべての値でログインが成功します。

### 4. **`$where`を利用したJavaScriptインジェクション**

**概要:**
MongoDBは、`$where`演算子を使用してJavaScriptコードを実行するクエリをサポートします。攻撃者はこれを利用して、任意のコードを実行するか、条件を回避することができます。

**脆弱性を持つコードの例:**

- **Node.js (ExpressとMongoDB)**

```javascript
const express = require('express');
const MongoClient = require('mongodb').MongoClient;

const app = express();
const url = 'mongodb://localhost:27017';
const dbName = 'mydatabase';

MongoClient.connect(url, (err, client) => {
    const db = client.db(dbName);
    const users = db.collection('users');

    app.get('/login', (req, res) => {
        const username = req.query.username;
        const password = req.query.password;

        users.findOne({ username: username, password: password }, (err, user) => {
            if (user) {
                res.send("Login successful!");
            } else {
                res.send("Login failed!");
            }
        });
    });

    app.listen(3000);
});
```

**攻撃手法:**

攻撃者は、`$where`に任意のJavaScriptコードを挿入してクエリを操作します。

例:
```
/login?username=admin&password[$where]=this.password.length>1
```

これにより、クエリは次のようになります:
```javascript
{ "username": "admin", "$where": "this.password.length>1" }
```

このクエリは、`username`が`admin`であれば、`password`の長さが1文字以上であればログインが成功します。

### 防止策

1. **パラメータ化クエリの使用**:
   - クエリにユーザー入力を直接挿入せず、パラメータ化されたクエリを使用してインジェクション攻撃を防ぎます。

2. **ユーザー入力の検証とサニタイズ**:
   - ユーザーからの入力データを常に検証し、不正な構文や操作を含むデータを拒否します。

3. **ホワイトリストによるフィールド検証**:
   - クエリで使用するフィールドや演算子をホワイトリストに基づいて検証し、許可されたフィールドや演算子のみを使用できるようにします。

4. **$whereや他のJavaScript演算子の使用を避ける**:
   - 可能な限り、`$where`演算子やその他のJavaScriptコードを実行する演算子を使用しないようにし、必要な場合は厳重に検証します。

5. **ORM/ODMの使用**:
   - オブジェクトリレーショナルマッピング（ORM）やオブジェクトドキュメントマッピング（ODM）を利用することで、クエリを直接作成することなく、安全にデータ操作を行います。

### まとめ

NoSQLインジェクションには、`$ne`以外にも、`$or`、`$regex`、`$gt`/`$lt`、`$where`などを利用

したさまざまな攻撃手法があります。これらの攻撃を防ぐためには、ユーザー入力の厳密な検証とサニタイズ、パラメータ化クエリの使用、そして不正なクエリ演算子の使用を制限することが重要です。また、ORM/ODMを利用することで、安全なクエリを実行することが可能になります。