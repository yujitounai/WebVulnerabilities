### JSONPハイジャックについて

#### 概要

**JSONPハイジャック**は、JSONP（JSON with Padding）というデータ交換形式を利用した攻撃手法です。JSONPは、クロスドメインでのデータ取得を容易にするために使われる手法で、サーバーからJSONデータを返す代わりに、クライアントが指定したコールバック関数を含むJavaScriptコードを返すというものです。攻撃者は、ユーザーの認証情報を持ったブラウザから、攻撃者が制御するドメインでこのJSONPレスポンスを利用してデータを取得することで、機密情報を盗むことができます。

#### 脆弱性を持つコードの例と攻撃手法

言語ごとの差異はないため、一般的な実装例と攻撃手法をまとめて説明します。

##### PHP

**脆弱性を持つコードの例:**

```php
<?php
$callback = $_GET['callback'];
$data = array('username' => 'john_doe', 'email' => 'john@example.com');
header('Content-Type: application/javascript');
echo $callback . '(' . json_encode($data) . ');';
```

##### Python (Flask)

**脆弱性を持つコードの例:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/user')
def user():
    callback = request.args.get('callback')
    data = {'username': 'john_doe', 'email': 'john@example.com'}
    response = app.response_class(
        response=f"{callback}({jsonify(data).get_data(as_text=True)})",
        status=200,
        mimetype='application/javascript'
    )
    return response
```

##### Node.js (Express)

**脆弱性を持つコードの例:**

```javascript
const express = require('express');
const app = express();

app.get('/user', (req, res) => {
    const callback = req.query.callback;
    const data = { username: 'john_doe', email: 'john@example.com' };
    res.type('application/javascript');
    res.send(`${callback}(${JSON.stringify(data)})`);
});

app.listen(3000);
```

##### Java (Servlet)

**脆弱性を持つコードの例:**

```java
import javax.servlet.http.*;
import java.io.IOException;

public class JsonpServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String callback = request.getParameter("callback");
        String jsonData = "{\"username\": \"john_doe\", \"email\": \"john@example.com\"}";
        response.setContentType("application/javascript");
        response.getWriter().write(callback + "(" + jsonData + ");");
    }
}
```

**攻撃手法:**

攻撃者は被害者を自身が制御するウェブページに誘導し、以下のようなスクリプトを実行させることで、JSONPレスポンスを不正に取得することができます。

```html
<!DOCTYPE html>
<html>
<head>
    <title>JSONP Hijacking</title>
</head>
<body>
    <script>
        function stealData(data) {
            console.log(data);
            // ここで攻撃者のサーバーにデータを送信するなどの操作を行う
        }

        var script = document.createElement('script');
        script.src = 'https://vulnerable-site.com/user?callback=stealData';
        document.body.appendChild(script);
    </script>
</body>
</html>
```

上記のスクリプトが実行されると、`vulnerable-site.com`のJSONPエンドポイントからデータが取得され、`stealData`関数に渡されます。攻撃者はこの関数を利用して、機密データを自身のサーバーに送信することができます。

#### 防止策

1. **JSONPの使用を避ける**:
   - 可能であれば、JSONPの使用を避け、代わりにCORS (Cross-Origin Resource Sharing) を利用したセキュアなデータ取得方法を採用します。

2. **コールバック関数名のホワイトリスト化**:
   - コールバック関数名をあらかじめ決められたホワイトリストに基づいて検証し、未知のコールバック関数を拒否するようにします。

   **例**:
   ```php
   <?php
   $allowed_callbacks = array('processData');
   $callback = $_GET['callback'];
   if (!in_array($callback, $allowed_callbacks)) {
       $callback = 'processData';  // デフォルトのコールバック関数
   }
   $data = array('username' => 'john_doe', 'email' => 'john@example.com');
   header('Content-Type: application/javascript');
   echo $callback . '(' . json_encode($data) . ');';
   ```

3. **JSONPのレスポンスに認証情報を含めない**:
   - JSONPのレスポンスに機密データや認証が必要な情報を含めないようにします。認証が必要なデータは、セキュアな手法でのみ提供するようにします。

4. **CSRFトークンの使用**:
   - JSONPリクエストに対しても、CSRFトークンを使用してリクエストの正当性を確認します。

5. **Strict Content Security Policy (CSP) の使用**:
   - CSPを利用して、信頼できるスクリプトソースのみを許可し、外部スクリプトが勝手に実行されないようにします。

6. **JSONPのサポートを停止する**:
   - サーバー側でJSONPをサポートしないようにし、CORSを使用したセキュアなデータ取得方法を推奨します。

### まとめ

JSONPハイジャックは、JSONPの仕組みを悪用して機密情報を盗み出す攻撃手法です。攻撃者はユーザーのブラウザを利用して不正にデータを取得し、自身のサーバーに送信することが可能です。JSONPの使用を避け、CORSを利用することが最も効果的な防御策ですが、やむを得ずJSONPを使用する場合は、コールバック関数のホワイトリスト化やCSRFトークンの使用など、適切なセキュリティ対策を講じることが重要です。