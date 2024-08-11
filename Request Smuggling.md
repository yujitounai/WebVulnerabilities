---
title: Request Smuggling
---

**Request Smuggling攻撃について**

### **概要**

Request Smuggling（リクエストスマグリング）攻撃は、HTTPリクエストが複数のサーバー（特にフロントエンドのプロキシサーバーとバックエンドのウェブサーバー）を経由する構成において発生します。この攻撃は、異なるサーバー間でHTTPリクエストの解釈に不一致がある場合に、攻撃者が悪意のあるリクエストを密かに送信し、予期しない動作や他のリクエストに対する干渉を引き起こすことを狙います。これにより、キャッシュポイズニング、セッションハイジャック、データ漏洩などが可能になります。

---

### **脆弱性を持つコードの例とサーバー構成と攻撃手法**

#### **サーバー構成**

**典型的な構成:**
- **フロントエンドプロキシ:** 例えば、Apache、Nginx、またはCDN（Content Delivery Network）など。
- **バックエンドサーバー:** アプリケーションサーバー（例えば、PHP、Python、Node.js、Javaなどの環境で動作する）。

フロントエンドプロキシは、HTTPリクエストを受け取り、バックエンドサーバーにそれを転送します。リクエストスマグリング攻撃は、この転送の際に異なるサーバーがHTTPヘッダーの解析を異なる方法で行うことを悪用します。

#### **攻撃手法の例:**
攻撃者は、`Content-Length`ヘッダーや`Transfer-Encoding`ヘッダーに細工することで、フロントエンドプロキシとバックエンドサーバーが同一のリクエストを異なる解釈をするように仕向けます。

**例1: 混在する`Content-Length`と`Transfer-Encoding`**
```http
POST / HTTP/1.1
Host: example.com
Content-Length: 13
Transfer-Encoding: chunked

0

SMUGGLEDREQUEST
```

**攻撃の流れ:**
1. フロントエンドプロキシがこのリクエストを受け取り、`Transfer-Encoding: chunked`を無視して、`Content-Length: 13`に従う場合、リクエストの最初の13バイトのみをバックエンドに転送します。
2. バックエンドサーバーは、`Transfer-Encoding: chunked`に従い、リクエスト全体を別のリクエストとして処理します。この結果、`SMUGGLEDREQUEST`が別のHTTPリクエストとして密かに送信され、通常のリクエストに混入します。

#### **PHP**

**脆弱なコード例:**
```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    echo "Request received!";
}
?>
```

**攻撃手法:**
このPHPコードはPOSTリクエストを単純に受け取って処理しますが、サーバー構成により、攻撃者は`SMUGGLEDREQUEST`をバックエンドサーバーに密かに送信し、他のリクエストに干渉する可能性があります。

#### **Python（Flask）**

**脆弱なコード例:**
```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/', methods=['POST'])
def index():
    return "Request received!"
```

**攻撃手法:**
Flaskアプリケーションも同様に、フロントエンドとバックエンドサーバーのリクエスト解析の不一致を突いて、攻撃者がリクエストスマグリングを行う可能性があります。

#### **Node.js**

**脆弱なコード例:**
```javascript
const express = require('express');
const app = express();

app.post('/', (req, res) => {
    res.send('Request received!');
});

app.listen(3000);
```

**攻撃手法:**
Node.jsとExpress.jsで構成されたアプリケーションに対しても、リクエストスマグリングを行うことで、悪意のあるリクエストが密かに処理されるリスクがあります。

#### **Java（Spring Boot）**

**脆弱なコード例:**
```java
import org.springframework.web.bind.annotation.*;

@RestController
public class MyController {

    @PostMapping("/")
    public String handlePostRequest() {
        return "Request received!";
    }
}
```

**攻撃手法:**
JavaのSpring Bootアプリケーションも、同様のサーバー構成でリクエストスマグリング攻撃を受ける可能性があります。

---

### **防止策**

1. **`Content-Length`と`Transfer-Encoding`の一貫性をチェック:**
   - フロントエンドプロキシとバックエンドサーバーで、`Content-Length`と`Transfer-Encoding`ヘッダーを一貫して処理するように設定します。例えば、`Transfer-Encoding: chunked`が含まれるリクエストは`Content-Length`ヘッダーを無視するか、エラーとして扱うようにします。

2. **フロントエンドプロキシでのリクエストサイズ制限:**
   - フロントエンドプロキシで、リクエストサイズやヘッダーの制限を厳格に設定することで、過剰なリクエストがバックエンドに到達するのを防ぎます。

3. **HTTPリクエストの正当性を検証:**
   - フロントエンドプロキシでリクエストの正当性を厳密に検証し、不正なヘッダーやリクエストフォーマットをブロックします。

4. **プロキシとバックエンドサーバーの設定を一致させる:**
   - フロントエンドプロキシとバックエンドサーバーが同一の設定でリクエストを処理するようにし、解釈の不一致が発生しないようにします。

5. **アップデートとパッチ適用:**
   - 使用しているプロキシサーバーやバックエンドサーバーソフトウェアが最新の状態であることを確認し、セキュリティパッチを適用して脆弱性を修正します。

---

Request Smuggling攻撃は、HTTPリクエストの解釈の違いを悪用した攻撃であり、フロントエンドプロキシとバックエンドサーバーの連携が重要です。適切な設定とセキュリティ対策を講じることで、この脆弱性を効果的に防ぐことが可能です。