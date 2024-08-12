---
title: CORSに関連する攻撃手法
---

**CORSに関連する攻撃手法について**

#### 概要

CORS（Cross-Origin Resource Sharing）は、ブラウザが異なるオリジン（異なるドメイン、プロトコル、またはポート）からリソースを安全に共有できるようにする仕組みです。CORSポリシーの設定が適切でない場合、悪意のある攻撃者が異なるオリジンからのリクエストを送信し、ユーザーの機密情報やデータを取得することができます。

CORSに関連する攻撃手法の主なものとして、以下が挙げられます。

1. **ワイルドカード (`*`) の誤使用**
2. **許可リストの不適切な設定**
3. **`Access-Control-Allow-Credentials` の誤使用**

### 脆弱性を持つコードの例と攻撃手法

言語による違いはほとんどないため、各言語の例を一つにまとめて説明します。

#### 1. ワイルドカード (`*`) の誤使用

**脆弱性を持つコードの例:**

```php
// PHP
header("Access-Control-Allow-Origin: *");
```

```python
# Python (Flask)
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/data')
def data():
    response = jsonify({"key": "value"})
    response.headers.add("Access-Control-Allow-Origin", "*")
    return response
```

```javascript
// Node.js (Express)
const express = require('express');
const app = express();

app.use((req, res, next) => {
    res.setHeader('Access-Control-Allow-Origin', '*');
    next();
});

app.get('/data', (req, res) => {
    res.json({ key: 'value' });
});

app.listen(3000);
```

```java
// Java (Servlet)
import javax.servlet.http.*;
import java.io.IOException;

public class CorsServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setContentType("application/json");
        response.getWriter().write("{\"key\": \"value\"}");
    }
}
```

**攻撃手法:**

`Access-Control-Allow-Origin: *` と設定された場合、任意のオリジンからのリクエストが許可されます。これにより、攻撃者は被害者のブラウザを利用して、任意のデータを取得し、自身のオリジンに送信することが可能になります。

例えば、攻撃者のサイトで以下のスクリプトを実行することで、他のオリジンから機密データを取得することができます。

```javascript
fetch('https://vulnerable-site.com/data')
    .then(response => response.json())
    .then(data => {
        console.log(data);
        // ここで攻撃者のサーバーにデータを送信するなどの操作を行う
    });
```

#### 2. 許可リストの不適切な設定

**脆弱性を持つコードの例:**

```php
// PHP
$origin = $_SERVER['HTTP_ORIGIN'];
$allowed_origins = ['https://trusteddomain.com'];
if (in_array($origin, $allowed_origins)) {
    header("Access-Control-Allow-Origin: $origin");
}
```

```python
# Python (Flask)
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/data')
def data():
    origin = request.headers.get('Origin')
    allowed_origins = ['https://trusteddomain.com']
    if origin in allowed_origins:
        response = jsonify({"key": "value"})
        response.headers.add("Access-Control-Allow-Origin", origin)
        return response
    return "Not allowed", 403
```

```javascript
// Node.js (Express)
const express = require('express');
const app = express();

const allowedOrigins = ['https://trusteddomain.com'];

app.use((req, res, next) => {
    const origin = req.headers.origin;
    if (allowedOrigins.includes(origin)) {
        res.setHeader('Access-Control-Allow-Origin', origin);
    }
    next();
});

app.get('/data', (req, res) => {
    res.json({ key: 'value' });
});

app.listen(3000);
```

```java
// Java (Servlet)
import javax.servlet.http.*;
import java.io.IOException;

public class CorsServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String origin = request.getHeader("Origin");
        if ("https://trusteddomain.com".equals(origin)) {
            response.setHeader("Access-Control-Allow-Origin", origin);
        }
        response.setContentType("application/json");
        response.getWriter().write("{\"key\": \"value\"}");
    }
}
```

**攻撃手法:**

攻撃者が`https://trusteddomain.com`を模倣するサイトを作成し、それを使ってリクエストを送信することで、データを不正に取得できる可能性があります。

例: `https://malicious-site.com`が被害者を誘導し、`trusteddomain.com`のクッキーを利用して機密データを取得する。

#### 3. `Access-Control-Allow-Credentials` の誤使用

**脆弱性を持つコードの例:**

```php
// PHP
header("Access-Control-Allow-Origin: https://trusteddomain.com");
header("Access-Control-Allow-Credentials: true");
```

```python
# Python (Flask)
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/data')
def data():
    response = jsonify({"key": "value"})
    response.headers.add("Access-Control-Allow-Origin", "https://trusteddomain.com")
    response.headers.add("Access-Control-Allow-Credentials", "true")
    return response
```

```javascript
// Node.js (Express)
const express = require('express');
const app = express();

app.use((req, res, next) => {
    res.setHeader('Access-Control-Allow-Origin', 'https://trusteddomain.com');
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    next();
});

app.get('/data', (req, res) => {
    res.json({ key: 'value' });
});

app.listen(3000);
```

```java
// Java (Servlet)
import javax.servlet.http.*;
import java.io.IOException;

public class CorsServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        response.setHeader("Access-Control-Allow-Origin", "https://trusteddomain.com");
        response.setHeader("Access-Control-Allow-Credentials", "true");
        response.setContentType("application/json");
        response.getWriter().write("{\"key\": \"value\"}");
    }
}
```

**攻撃手法:**

`Access-Control-Allow-Credentials: true` が設定されていると、クッキーなどの認証情報が含まれたリクエストが許可されます。攻撃者はこれを利用して、認証された状態で他のオリジンに対するリクエストを送信し、その結果を自身のオリジンに取得することが可能になります。

### 防止策

1. **ワイルドカード (`*`) の使用を避ける**: `Access-Control-Allow-Origin`にワイルドカードを使うと、すべてのオリジンからのリクエストが許可されます。これを避け、信頼できる特定のオリジンだけを許可するように設定します。

2. **動的にオリジンを設定する場合の安全性を確認**: 許可されたオリジンのみが`Access-Control-Allow-Origin`ヘッダーに設定されるように、信頼できるオリジンのリストを使用して、適切にオリジンを検証します。

3. **`Access-Control-Allow-Credentials` の使用を慎重に行う**: このヘッダーを使用する場合は、特定のオリジンに対してのみ許可し、かつ本当に必要な場合に限るようにします。ワイルドカードと一緒に使用してはいけません。

4. **レスポンスヘッダーの見直し**: 必要なCORSヘッダーが正しく設定されているか、不要なCORSヘッダーが設定されていないか確認します。

5. **APIエンドポイントの適切な設計**: 特に認証が必要なエンドポイントに対して、セキュリティの高い設計を行い、認証が不要なエンドポイントではCORSを利用しない、または利用を最小限に抑えることが重要です。

### まとめ

CORSに関連する脆弱性は、CORSポリシーの誤設定に起因することが多いです。これにより、攻撃者は異なるオリジンからのリクエストを利用して、機密データを取得

することが可能になります。防止策としては、CORS設定を慎重に行い、特に`Access-Control-Allow-Origin`や`Access-Control-Allow-Credentials`の設定を見直すことが重要です。また、APIエンドポイントの設計段階からセキュリティを考慮することが求められます。