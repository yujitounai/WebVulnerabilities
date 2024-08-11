**XSS（クロスサイトスクリプティング）攻撃について**

### **概要**

XSS（Cross-Site Scripting）は、Webアプリケーションの脆弱性を利用して、攻撃者がユーザーのブラウザ上で悪意のあるスクリプトを実行させる攻撃手法です。これにより、ユーザーのセッションを乗っ取ったり、クッキーを盗んだり、フィッシング詐欺を仕掛けたりすることができます。XSSは、特にユーザー入力を適切に処理しないWebアプリケーションにおいて発生しやすく、攻撃者は様々な方法でXSSを仕掛けることができます。主に、次の3つの種類に分類されます。

- **Stored XSS（蓄積型XSS）**: 悪意のあるスクリプトがサーバーに保存され、他のユーザーがページを閲覧した際にスクリプトが実行される。
- **Reflected XSS（反射型XSS）**: 悪意のあるスクリプトがリクエストに含まれ、そのまま応答に反映されてブラウザで実行される。
- **DOM-based XSS**: クライアント側で実行されるスクリプトによってDOMが動的に操作され、悪意のあるスクリプトが実行される。

---

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP**

**脆弱なコード例（Reflected XSS）:**
```php
<?php
$name = $_GET['name'];
echo "Hello, " . $name . "!";
?>
```

**攻撃手法:**
攻撃者は、以下のようなURLを犠牲者にクリックさせることで、XSS攻撃を実行できます。

```bash
http://example.com/?name=<script>alert('XSS')</script>
```

この場合、スクリプトがページ内に埋め込まれてブラウザで実行され、アラートが表示されます。

#### **Python（Flask）**

**脆弱なコード例（Stored XSS）:**
```python
from flask import Flask, request, render_template_string

app = Flask(__name__)
comments = []

@app.route('/comment', methods=['POST'])
def comment():
    comment = request.form['comment']
    comments.append(comment)
    return "Comment added!"

@app.route('/comments')
def view_comments():
    return render_template_string("<ul>{% for comment in comments %}<li>{{ comment }}</li>{% endfor %}</ul>", comments=comments)

if __name__ == '__main__':
    app.run()
```

**攻撃手法:**
攻撃者が以下のようなコメントを投稿すると、他のユーザーがページを閲覧する際にXSS攻撃が実行されます。

```html
<script>alert('XSS')</script>
```

#### **Node.js（Express）**

**脆弱なコード例（Reflected XSS）:**
```javascript
const express = require('express');
const app = express();

app.get('/greet', (req, res) => {
    const name = req.query.name;
    res.send(`Hello, ${name}!`);
});

app.listen(3000);
```

**攻撃手法:**
攻撃者は、以下のようなURLを使用してXSS攻撃を実行できます。

```bash
http://localhost:3000/greet?name=<script>alert('XSS')</script>
```

#### **Java（Spring）**

**脆弱なコード例（Reflected XSS）:**
```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.stereotype.Controller;

@Controller
public class GreetingController {

    @GetMapping("/greet")
    public String greet(@RequestParam String name, Model model) {
        model.addAttribute("message", "Hello, " + name + "!");
        return "greet";
    }
}
```

**攻撃手法:**
攻撃者は、以下のようなURLを使用してXSS攻撃を実行できます。

```bash
http://localhost:8080/greet?name=<script>alert('XSS')</script>
```

---

### **防止策**

1. **出力時のエスケープ処理:**
   - ユーザー入力をHTMLに出力する際は、必ずエスケープ処理を行い、特殊文字がスクリプトとして解釈されないようにします。

   **PHPの例**:
   ```php
   echo "Hello, " . htmlspecialchars($name, ENT_QUOTES, 'UTF-8') . "!";
   ```

   **Pythonの例（Flask）**:
   Flaskのテンプレートエンジン（Jinja2）はデフォルトでエスケープ処理が行われますが、テンプレートの使用時には慎重に扱うべきです。

   **Node.jsの例**:
   ```javascript
   const escapeHtml = require('escape-html');
   res.send(`Hello, ${escapeHtml(name)}!`);
   ```

   **Javaの例（Spring）**:
   ThymeleafやJSPのテンプレートエンジンでは、デフォルトでエスケープ処理が行われます。

2. **入力のサニタイズ:**
   - ユーザー入力を受け取る際に、スクリプトやHTMLタグを除去するためのサニタイズを行います。

   **PHPの例**:
   ```php
   $name = strip_tags($_GET['name']);
   ```

   **Pythonの例（Flask）**:
   ```python
   from bleach import clean
   comment = clean(request.form['comment'])
   ```

3. **CSP（コンテンツセキュリティポリシー）の設定:**
   - CSPヘッダーを使用して、許可されたスクリプトソースのみが実行されるように制限します。

   **PHPの例**:
   ```php
   header("Content-Security-Policy: default-src 'self'");
   ```

   **Node.jsの例**:
   ```javascript
   app.use((req, res, next) => {
       res.setHeader("Content-Security-Policy", "default-src 'self'");
       next();
   });
   ```

4. **セキュリティライブラリの使用:**
   - 可能であれば、XSS対策が組み込まれたセキュリティライブラリやフレームワークを使用することが推奨されます。

5. **ユーザー入力の検証と制限:**
   - 入力の形式や長さを制限し、予期しないスクリプトが入力されることを防ぎます。特に、不要なHTMLやスクリプトタグを含む入力を拒否します。

6. **HttpOnlyとSecureクッキーの設定:**
   - クッキーに`HttpOnly`属性を設定し、JavaScriptからのアクセスを防ぎます。また、`Secure`属性を設定して、HTTPS経由でのみクッキーが送信されるようにします。

---

**まとめ:**
XSS攻撃は、Webアプリケーションが適切なエスケープ処理やサニタイズを行わない場合に発生する脆弱性です。出力時のエスケープ、入力のサニタイズ、CSPの設定、セキュリティライブラリの使用などの防止策を実施することで、この脆弱性を効果的に防ぐことができます。各プラットフォームや言語に共通する問題であるため、これらの対策を適用することが推奨されます。

#### **Python（Flask） - 反射型XSSの例**

**脆弱なコード例（Reflected XSS）:**
```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/greet')
def greet():
    name = request.args.get('name')
    return f"Hello, {name}!"

if __name__ == '__main__':
    app.run()
```

**攻撃手法:**
攻撃者が以下のようなURLを使用してXSS攻撃を実行できます。

```bash
http://localhost:5000/greet?name=<script>alert('XSS')</script>
```

このURLをクリックすると、ブラウザ上でスクリプトが実行され、アラートが表示されます。

---

### **防止策**

**出力時のエスケープ処理:**
- ユーザー入力をHTMLに出力する際には、必ずエスケープ処理を行い、特殊文字がスクリプトとして解釈されないようにします。

**修正されたコード例:**
```python
from flask import Flask, request
from markupsafe import escape

app = Flask(__name__)

@app.route('/greet')
def greet():
    name = request.args.get('name')
    safe_name = escape(name)  # 入力をエスケープ
    return f"Hello, {safe_name}!"

if __name__ == '__main__':
    app.run()
```

**`escape`関数**は、ユーザーの入力をHTMLで安全に表示できるようにエスケープ処理を行います。これにより、`<script>`タグなどの特殊文字がそのまま表示され、スクリプトとして実行されることを防ぎます。

この対策により、攻撃者が`<script>`タグを使用しても、それが文字列として扱われ、スクリプトが実行されることはなくなります。