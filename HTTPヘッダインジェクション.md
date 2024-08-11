---
title: HTTPヘッダインジェクション
---

### HTTPヘッダインジェクション/HTTP Request Header Injection について

#### 概要
HTTPヘッダインジェクションは、アプリケーションがユーザーからの入力をHTTPリクエストのヘッダーに直接組み込む際に発生する脆弱性です。この脆弱性を利用することで、攻撃者はヘッダーフィールドに改ざんや追加を行い、サーバーの動作を予期しないものにしたり、ユーザーを別のサイトにリダイレクトさせたりすることが可能です。主なリスクには、セッションの乗っ取り、クロスサイトスクリプティング (XSS) の実行、レスポンススプリッティング攻撃などが含まれます。

#### 脆弱性を持つコードの例と攻撃手法

##### PHP
**脆弱性を持つコードの例:**
```php
<?php
$user_agent = $_GET['user_agent'];
header("User-Agent: " . $user_agent);
```
**攻撃手法:**
- URLに`http://example.com/?user_agent=foo%0D%0ALocation:%20http://malicious.com`と入力すると、攻撃者はリクエストヘッダーに新しいヘッダー行を挿入し、ユーザーを`http://malicious.com`にリダイレクトさせることが可能です。

##### Python (Flask)
**脆弱性を持つコードの例:**
```python
from flask import Flask, request, make_response

app = Flask(__name__)

@app.route('/')
def index():
    user_agent = request.args.get('user_agent')
    response = make_response("Hello World")
    response.headers['User-Agent'] = user_agent
    return response
```
**攻撃手法:**
- URLに`http://example.com/?user_agent=foo%0D%0ALocation:%20http://malicious.com`と入力すると、`User-Agent`ヘッダーの後に新しいヘッダー行を追加し、レスポンスヘッダーを操作できます。

##### Node.js (Express)
**脆弱性を持つコードの例:**
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    const userAgent = req.query.user_agent;
    res.set('User-Agent', userAgent);
    res.send('Hello World');
});

app.listen(3000);
```
**攻撃手法:**
- URLに`http://example.com/?user_agent=foo%0D%0ALocation:%20http://malicious.com`を使うことで、攻撃者はヘッダーを改ざんし、別の場所にリダイレクトさせることが可能です。

##### Java (Servlet)
**脆弱性を持つコードの例:**
```java
import javax.servlet.http.*;
import java.io.IOException;

public class HeaderInjectionExample extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String userAgent = request.getParameter("user_agent");
        response.setHeader("User-Agent", userAgent);
        response.getWriter().write("Hello World");
    }
}
```
**攻撃手法:**
- URLに`http://example.com/?user_agent=foo%0D%0ALocation:%20http://malicious.com`と入力することで、Java Servletでも同様にヘッダーの改ざんが可能です。

#### 防止策
各言語において、HTTPヘッダインジェクションを防ぐための一般的な対策は共通しており、以下のような方法があります。

1. **ユーザー入力の検証とサニタイズ**:
   - ヘッダーに含まれる特殊文字（例えば、CR (Carriage Return) `\r`やLF (Line Feed) `\n`）を除去またはエスケープすることで、ヘッダーの改ざんを防止します。
   
2. **既知の安全な形式でのヘッダー設定**:
   - ヘッダーにユーザー入力を直接使用するのではなく、予め定義された安全な値の中から選択して設定するようにします。

3. **ライブラリやフレームワークのセキュリティ機能の利用**:
   - 使用するフレームワークやライブラリに組み込まれているセキュリティ機能を活用し、ヘッダーインジェクションのリスクを軽減します。

##### 防止策の例:

- **PHP**:
```php
<?php
$user_agent = filter_input(INPUT_GET, 'user_agent', FILTER_SANITIZE_STRING);
header("User-Agent: " . $user_agent);
```

- **Python (Flask)**:
```python
from flask import Flask, request, make_response
import re

app = Flask(__name__)

@app.route('/')
def index():
    user_agent = request.args.get('user_agent')
    if user_agent:
        user_agent = re.sub(r'[\r\n]', '', user_agent)
    response = make_response("Hello World")
    response.headers['User-Agent'] = user_agent
    return response
```

- **Node.js (Express)**:
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    let userAgent = req.query.user_agent;
    userAgent = userAgent.replace(/[\r\n]/g, '');  // CRとLFを除去
    res.set('User-Agent', userAgent);
    res.send('Hello World');
});

app.listen(3000);
```

- **Java (Servlet)**:
```java
import javax.servlet.http.*;
import java.io.IOException;

public class HeaderInjectionExample extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String userAgent = request.getParameter("user_agent");
        if (userAgent != null) {
            userAgent = userAgent.replaceAll("[\\r\\n]", "");  // CRとLFを除去
        }
        response.setHeader("User-Agent", userAgent);
        response.getWriter().write("Hello World");
    }
}
```

### まとめ
HTTPヘッダインジェクションは、ユーザー入力が適切に検証されずにHTTPヘッダーに組み込まれる場合に発生する脆弱性です。攻撃者はこれを利用して、予期しないヘッダー操作やリダイレクト、セッション乗っ取りなどの攻撃を行う可能性があります。防止策としては、ユーザー入力の検証とサニタイズが最も重要です。また、各言語で提供されるフレームワークやライブラリのセキュリティ機能を活用することで、この脆弱性を効果的に防ぐことができます。