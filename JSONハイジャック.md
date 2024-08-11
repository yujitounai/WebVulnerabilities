---
title: JSONハイジャック
---

**JSONハイジャックについて**

#### 概要

**JSONハイジャック (JSON Hijacking)** とは、攻撃者が他人のセッション内で実行されるJavaScriptを利用し、JSON形式で送信される機密情報を盗み出す攻撃手法です。この攻撃は主に、ウェブアプリケーションがユーザーの機密データをJSON形式で返し、それをブラウザで直接評価する場合に発生します。

特に、ブラウザが同一オリジンポリシーを厳格に適用しない場合や、古いブラウザが使われている場合に、この攻撃が成立する可能性が高まります。

### 脆弱性を持つコードの例と攻撃手法

言語に依存しない一般的な問題として、JSONハイジャックは以下のような状況で発生します。

**脆弱性を持つコードの例:**

仮に以下のようなAPIエンドポイントがあったとします。

```php
<?php
header('Content-Type: application/json');
$data = array('username' => 'john_doe', 'email' => 'john@example.com');
echo json_encode($data);
```

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/user')
def user():
    data = {'username': 'john_doe', 'email': 'john@example.com'}
    return jsonify(data)
```

```javascript
const express = require('express');
const app = express();

app.get('/user', (req, res) => {
    res.json({ username: 'john_doe', email: 'john@example.com' });
});

app.listen(3000);
```

```java
import javax.servlet.http.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.IOException;

public class UserServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("application/json");
        ObjectMapper mapper = new ObjectMapper();
        User user = new User("john_doe", "john@example.com");
        mapper.writeValue(response.getOutputStream(), user);
    }
}

class User {
    public String username;
    public String email;

    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
}
```

**攻撃手法:**

攻撃者が次のような悪意のあるページを用意します:

```html
<!DOCTYPE html>
<html>
<head>
    <title>JSON Hijacking</title>
</head>
<body>
    <script>
        window.onload = function() {
            var script = document.createElement('script');
            script.src = 'https://example.com/user'; // これは脆弱なAPIエンドポイントです
            document.body.appendChild(script);
        }
    </script>
</body>
</html>
```

このページが被害者のブラウザで開かれると、APIエンドポイントからJSONデータが読み込まれ、`script`タグとして評価されます。JSONデータがJavaScriptのコードとして実行されるため、攻撃者は`script.src`に指定したエンドポイントから機密情報を取得できます。

### 防止策

#### 1. **JSONラッピング**
JSONレスポンスを配列ではなくオブジェクトにラップする方法です。JavaScriptは配列のJSONをそのまま評価してしまいますが、オブジェクトを直接評価することはできません。

**例:**

```php
<?php
header('Content-Type: application/json');
$data = array('username' => 'john_doe', 'email' => 'john@example.com');
echo 'while(1);' . json_encode($data);
```

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/user')
def user():
    data = {'username': 'john_doe', 'email': 'john@example.com'}
    return 'while(1);' + jsonify(data).get_data(as_text=True)
```

```javascript
const express = require('express');
const app = express();

app.get('/user', (req, res) => {
    res.set('Content-Type', 'application/json');
    res.send('while(1);' + JSON.stringify({ username: 'john_doe', email: 'john@example.com' }));
});

app.listen(3000);
```

```java
import javax.servlet.http.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.IOException;

public class UserServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("application/json");
        response.getOutputStream().print("while(1);");
        ObjectMapper mapper = new ObjectMapper();
        User user = new User("john_doe", "john@example.com");
        mapper.writeValue(response.getOutputStream(), user);
    }
}
```

#### 2. **CSRFトークンの使用**
JSONハイジャック攻撃は、CSRF（クロスサイトリクエストフォージェリ）トークンを使用することで防ぐことができます。リクエストに有効なトークンが含まれていない場合、サーバーはレスポンスを返さないようにします。

#### 3. **HTTPヘッダーを使った防御**
`X-Content-Type-Options: nosniff` ヘッダーを使用し、ブラウザがスクリプトとして誤ってコンテンツを解釈しないようにします。また、APIエンドポイントで`Content-Disposition: attachment` ヘッダーを設定し、コンテンツがファイルとして扱われるようにすることも有効です。

**例:**

```php
<?php
header('Content-Type: application/json');
header('X-Content-Type-Options: nosniff');
$data = array('username' => 'john_doe', 'email' => 'john@example.com');
echo json_encode($data);
```

```python
from flask import Flask, jsonify, make_response

app = Flask(__name__)

@app.route('/user')
def user():
    data = {'username': 'john_doe', 'email': 'john@example.com'}
    response = make_response(jsonify(data))
    response.headers['X-Content-Type-Options'] = 'nosniff'
    return response
```

```javascript
const express = require('express');
const app = express();

app.get('/user', (req, res) => {
    res.set('Content-Type', 'application/json');
    res.set('X-Content-Type-Options', 'nosniff');
    res.json({ username: 'john_doe', email: 'john@example.com' });
});

app.listen(3000);
```

```java
import javax.servlet.http.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.IOException;

public class UserServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setContentType("application/json");
        response.setHeader("X-Content-Type-Options", "nosniff");
        ObjectMapper mapper = new ObjectMapper();
        User user = new User("john_doe", "john@example.com");
        mapper.writeValue(response.getOutputStream(), user);
    }
}
```

### まとめ
JSONハイジャックは、特に古いブラウザや不適切な実装において、攻撃者がJSONデータを盗み出すことが可能になる脆弱性です。主な防止策には、JSONレスポンスのラッピング、CSRFトークンの導入、適切なHTTPヘッダーの使用などがあります。これらの対策を実装することで、JSONハイジャックのリスクを効果的に軽減することが可能です。

CORS（Cross-Origin Resource Sharing）の問題を利用してJSONデータが取得できる手法とJSONハイジャックは関連する部分があるものの、技術的には異なる攻撃手法です。以下にそれぞれの違いと関係性について説明します。

### CORSの問題によるJSONデータの取得

CORSの設定ミスや脆弱性を利用して、攻撃者が異なるオリジンからのリクエストに対して、許可されるべきでないデータを取得する攻撃です。

**概要:**
- CORSは、ブラウザが異なるオリジンからのリクエストを制御するための仕組みです。適切に設定されていない場合、攻撃者は被害者のブラウザを利用して、異なるオリジンからのデータを不正に取得することができます。
- これにより、攻撃者はユーザーの機密データを取得したり、ユーザーが意図しないアクションを実行させたりすることが可能になります。

**例:**
- 攻撃者のサイト（`malicious.com`）が、被害者のサイト（`example.com`）に対してAJAXリクエストを送信し、`example.com`からのレスポンスを取得できる場合、これはCORSの設定ミスが原因である可能性があります。

**CORSの問題が原因でJSONが取得されるシナリオ:**
- サーバーが誤って`Access-Control-Allow-Origin: *`や、特定のオリジン（攻撃者のオリジンを含む）に対して許可を与えている場合。
- サーバーが`Access-Control-Allow-Credentials: true`を設定し、さらに不適切なオリジンを許可している場合、クッキーや認証情報が含まれるリクエストが攻撃者のオリジンから送信され、レスポンスが攻撃者に返されることがあります。

### JSONハイジャック

**概要:**
- JSONハイジャックは、攻撃者が被害者のブラウザを利用して、JSON形式のデータを別のサイトから取得する手法です。主に、ブラウザがJSONをJavaScriptとして評価することを悪用します。

**関係性:**
- CORSの脆弱性とJSONハイジャックは、どちらもブラウザの制約（特にオリジンポリシー）を回避して機密データを取得する手法ですが、アプローチが異なります。
  - **CORSの問題**では、サーバーが異なるオリジンからのリクエストを許可する設定ミスを悪用します。
  - **JSONハイジャック**では、特に`<script>`タグを使ったリクエストを利用し、JSONレスポンスがJavaScriptコードとして評価されることを悪用します。

### 違いと関係性

- **CORSに関連する攻撃**は、サーバーの設定ミスを悪用して、他のオリジンからリクエストを送信し、そのレスポンスを攻撃者が利用するものです。
- **JSONハイジャック**は、主に古いブラウザやJSONデータがそのままJavaScriptのオブジェクトや配列として評価されることを悪用して、データを盗み出すものです。

ただし、CORS設定が適切でない場合に、攻撃者がJSONデータを取得する手段を得ることがあるため、結果としてJSONハイジャックと類似した状況を生み出す可能性があります。

### まとめ

CORSの設定ミスでJSONデータが取得されるケースとJSONハイジャックは、いずれもブラウザのオリジンポリシーを回避してデータを盗む攻撃手法ですが、技術的には異なるアプローチです。どちらの問題も、適切なセキュリティ対策を講じることで防ぐことができます。JSONハイジャックの場合は、JSONレスポンスの構造を工夫することや、CORS問題を防ぐためには適切なCORS設定を行うことが重要です。