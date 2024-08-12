---
title: CRLFインジェクション
---

**CRLFインジェクション攻撃について**

### **概要**

CRLFインジェクション攻撃とは、Webアプリケーションにおいて、ユーザーが入力したデータがHTTPレスポンスヘッダーや他のプロトコルのヘッダーにそのまま挿入される場合に発生する攻撃です。CRLFは、キャリッジリターン（`\r`）とラインフィード（`\n`）のことで、HTTPヘッダーを終了し、新しいヘッダーやボディを開始するために使われます。この攻撃により、攻撃者は任意のヘッダーを挿入したり、レスポンスのボディを改ざんすることができます。これにより、XSS（クロスサイトスクリプティング）やHTTPレスポンススプリッティングなどの攻撃が実行可能となります。

---

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP**

**脆弱なコード例:**
```php
<?php
$user = $_GET['user'];
header("Location: /welcome.php?user=" . $user);
?>
```

**攻撃手法:**
- 攻撃者が以下のようなリクエストを送信することで、HTTPレスポンスヘッダーを改ざんすることができます。

  ```bash
  http://example.com/vulnerable.php?user=John%0d%0aSet-Cookie:%20malicious=1
  ```

  このリクエストにより、`Set-Cookie`ヘッダーがレスポンスに追加され、クライアントに不正なクッキーが設定されます。

### **PHPのバージョンによる違い**

#### **PHP 5.xおよびそれ以前のバージョン**

- **脆弱性の高い状態**:
  - PHP 5.x以前のバージョンでは、`header()`関数に渡される入力があまり厳密に検証されておらず、ユーザー入力をそのままHTTPヘッダーに挿入すると、CRLFインジェクションが発生しやすい状況でした。
  - このため、攻撃者が`header()`関数を悪用して、HTTPレスポンスヘッダーを改ざんすることが可能でした。

- **例**:
  ```php
  <?php
  $user = $_GET['user'];
  header("Location: /home.php?user=" . $user);
  ?>
  ```
  - このコードでは、攻撃者が`user`パラメータに改行コード（`%0d%0a`）を挿入することで、HTTPヘッダーに追加のヘッダーを注入することが可能です。

#### **PHP 5.1.2以降**

- **対策の導入**:
  - PHP 5.1.2以降では、`header()`関数が入力に改行コード（`\r`や`\n`）を含むことを許可しないように改善されました。具体的には、ヘッダーに改行が含まれる場合、PHPが自動的に例外をスローし、攻撃を防ぐようになりました。
  - この変更により、CRLFインジェクションを防ぐためのセキュリティが強化され、意図せずにヘッダーが分割されることがなくなりました。

- **例**:
  ```php
  <?php
  $user = $_GET['user'];
  // PHP 5.1.2以降では、次のコードが改行を含む場合にエラーを発生させます。
  header("Location: /home.php?user=" . $user); 
  ?>
  ```
  - ここで、`user`パラメータに`%0d%0a`が含まれていると、PHPがエラーをスローし、インジェクションを防ぎます。

#### **PHP 7.x以降**

- **さらなるセキュリティ強化**:
  - PHP 7.x以降でも、ヘッダーの操作に対するセキュリティ対策がさらに強化され、CRLFインジェクションのリスクがさらに低減されました。
  - 現在のバージョンのPHPでは、`header()`関数でCRLFが検出された場合、PHPは警告を出し、そのヘッダーを送信しません。

- **例**:
  ```php
  <?php
  $user = $_GET['user'];
  header("Location: /home.php?user=" . $user); 
  ?>
  ```
  - PHP 7.x以降では、`user`に改行コードが含まれている場合、エラーが発生し、レスポンスが正しく処理されないようにしています。

#### **Python（Flask）**

**脆弱なコード例:**
```python
from flask import Flask, request, redirect

app = Flask(__name__)

@app.route('/redirect')
def redirect_user():
    user = request.args.get('user')
    return redirect(f'/welcome?user={user}')

if __name__ == '__main__':
    app.run()
```

**攻撃手法:**
- 攻撃者が以下のようなURLを使用して、レスポンスヘッダーを改ざんします。

  ```bash
  http://localhost:5000/redirect?user=John%0D%0AContent-Length:%200
  ```

  この攻撃により、レスポンスが分割され、不正な内容が挿入される可能性があります。

### **Pythonのバージョンによる差異**

#### **標準ライブラリ**

Pythonの標準ライブラリ自体には、HTTPヘッダーの生成や操作に特化した関数は含まれていません。代わりに、`http.client`モジュール（Python 3.x）や`httplib`モジュール（Python 2.x）を使用して、HTTPリクエストを作成したり、ヘッダーを操作することができます。CRLFインジェクションに関しては、Pythonの標準ライブラリ自体にバージョンによる大きな差異はありません。

- **Python 2.x**:
  - `httplib`モジュールを使用して、HTTPリクエストを生成します。CRLFインジェクションを防ぐためのバリデーションは標準で提供されていないため、開発者が入力をサニタイズする必要があります。

- **Python 3.x**:
  - `http.client`モジュールを使用します。Python 3.xでは、HTTPヘッダーを操作する際に、改行文字（`\r`や`\n`）がヘッダーに含まれていると例外が発生するようになっています。これにより、CRLFインジェクションのリスクは軽減されています。

#### **Webフレームワーク（FlaskやDjango）**

PythonでWebアプリケーションを構築する際には、FlaskやDjangoなどのWebフレームワークを使用することが一般的です。これらのフレームワークでは、ユーザー入力を適切にサニタイズしないと、CRLFインジェクションのリスクがあります。

- **Flask**:
  - Flaskでは、HTTPレスポンスヘッダーを手動で設定する場合、Pythonのバージョンに依存せずに、開発者が入力をサニタイズする必要があります。
  - Flask自体には、バージョンによるCRLFインジェクションの防止に特化した変更はありませんが、Python 3.xを使用することで、ヘッダー操作時のバリデーションが強化されます。

  **脆弱なコード例**:
  ```python
  from flask import Flask, request, make_response

  app = Flask(__name__)

  @app.route('/set_header')
  def set_header():
      user = request.args.get('user')
      response = make_response("Setting header")
      response.headers['X-User'] = user
      return response
  ```

  **攻撃手法**:
  - 攻撃者が以下のように改行を含む`user`パラメータを送信することで、ヘッダーに不正なデータを注入する可能性があります。

  ```bash
  http://example.com/set_header?user=admin%0d%0aX-Attack:1
  ```

  - Python 3.xでは、`http.client`モジュールが改行を含むヘッダーに対して例外を発生させるため、Flaskアプリケーションが例外を処理しない限り、この攻撃は失敗します。

- **Django**:
  - Djangoでは、バージョン1.8以降で、HTTPヘッダーに改行を含む値を設定しようとすると、自動的に例外が発生するようになっています。これにより、CRLFインジェクションのリスクが低減されています。

  **脆弱なコード例**:
  ```python
  from django.http import HttpResponse

  def set_header(request):
      user = request.GET.get('user', '')
      response = HttpResponse("Setting header")
      response['X-User'] = user
      return response
  ```

  **攻撃手法**:
  - Django 1.8以降のバージョンでは、以下のような攻撃を試みても例外が発生するため、ヘッダーインジェクションを防止できます。

  ```bash
  http://example.com/set_header?user=admin%0d%0aX-Attack:1
  ```

#### **Node.js（Express）**

**脆弱なコード例:**
```javascript
const express = require('express');
const app = express();

app.get('/redirect', (req, res) => {
    const user = req.query.user;
    res.setHeader('Location', `/welcome?user=${user}`);
    res.status(302).end();
});

app.listen(3000);
```

**攻撃手法:**
- 攻撃者は以下のURLを使用して、レスポンスヘッダーを改ざんします。

  ```bash
  http://localhost:3000/redirect?user=John%0d%0aContent-Length:%200
  ```

  これにより、HTTPレスポンスヘッダーが改ざんされ、予期しない動作を引き起こす可能性があります。



#### **Java（Spring）**

**脆弱なコード例:**
```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

import javax.servlet.http.HttpServletResponse;

@Controller
public class RedirectController {

    @GetMapping("/redirect")
    public void redirectUser(@RequestParam String user, HttpServletResponse response) {
        response.setHeader("Location", "/welcome?user=" + user);
        response.setStatus(HttpServletResponse.SC_FOUND);
    }
}
```

**攻撃手法:**
- 攻撃者が以下のようにリクエストを送信して、レスポンスヘッダーを改ざんします。

  ```bash
  http://localhost:8080/redirect?user=John%0D%0ASet-Cookie:%20malicious=1
  ```

  これにより、`Set-Cookie`ヘッダーが追加され、不正なクッキーが設定されます。

---

### **防止策**

1. **ユーザー入力のエスケープまたはサニタイズ**:
   - ユーザーが入力したデータをHTTPレスポンスヘッダーに挿入する前に、CRLF文字列（`\r`と`\n`）をエスケープまたは削除します。

   **PHPの例**:
   ```php
   $user = str_replace(["\r", "\n"], '', $_GET['user']);
   header("Location: /welcome.php?user=" . $user);
   ```

   **Python（Flask）の例**:
   ```python
   from flask import Flask, request, redirect
   import re

   app = Flask(__name__)

   @app.route('/redirect')
   def redirect_user():
       user = re.sub(r'[\r\n]', '', request.args.get('user'))
       return redirect(f'/welcome?user={user}')

   if __name__ == '__main__':
       app.run()
   ```

   **Node.js（Express）の例**:
   ```javascript
   const express = require('express');
   const app = express();

   app.get('/redirect', (req, res) => {
       const user = req.query.user.replace(/[\r\n]/g, '');
       res.setHeader('Location', `/welcome?user=${user}`);
       res.status(302).end();
   });

   app.listen(3000);
   ```

   **Java（Spring）の例**:
   ```java
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestParam;

   import javax.servlet.http.HttpServletResponse;

   @Controller
   public class RedirectController {

       @GetMapping("/redirect")
       public void redirectUser(@RequestParam String user, HttpServletResponse response) {
           user = user.replaceAll("[\\r\\n]", "");
           response.setHeader("Location", "/welcome?user=" + user);
           response.setStatus(HttpServletResponse.SC_FOUND);
       }
   }
   ```

2. **HTTPレスポンスヘッダーにユーザー入力を直接使用しない**:
   - 可能な限り、ユーザー入力をHTTPレスポンスヘッダーに直接使用しないように設計します。動的な値が必要な場合は、定義された範囲や形式の中でのみ使用するようにします。

3. **セキュリティフレームワークの利用**:
   - フレームワークやライブラリで提供されるセキュアなメソッドやコンポーネントを利用し、手動でヘッダーを構築しないようにします。これにより、セキュリティリスクを低減できます。

4. **セキュリティテストの実施**:
   - CRLFインジェクション脆弱性がないかを検証するため、セキュリティテストを実施し、コードが安全であることを確認します。

---

**まとめ**:
CRLFインジェクションは、Webアプリケーションがユーザー入力を適切に処理しない場合に発生する攻撃です。攻撃者は、この脆弱性を利用してHTTPレスポンスヘッダーを改ざんし、不正なクッキーの設定やレスポンスの改ざんなどを行うことができます。ユーザー入力のエスケープやサニタイズ、セキュリティフレームワークの利用などの防止策を講じることで、この脆弱性を防ぐことが可能です。


CRLFインジェクションとHTTPヘッダインジェクションは密接に関連している攻撃手法であり、両者はしばしば混同されることがありますが、それぞれが焦点を当てる部分には若干の違いがあります。

### **CRLFインジェクション**
**概要**:
- **CRLF**は「キャリッジリターン（`Carriage Return`, `\r`）」と「ラインフィード（`Line Feed`, `\n`）」を指します。これらは、HTTPプロトコルにおいてヘッダーフィールドを終了し、新しいヘッダーやボディの開始を示すために使用されます。
- **CRLFインジェクション**は、Webアプリケーションにおいて、ユーザーが提供した入力がそのままレスポンスのHTTPヘッダーに挿入され、かつその入力に`CRLF`文字（`\r\n`）が含まれている場合に発生する脆弱性です。これにより、攻撃者はレスポンスヘッダーを任意に操作し、新しいヘッダーを追加したり、レスポンスのボディを改ざんしたりすることが可能になります。

### **HTTPヘッダインジェクション**
**概要**:
- **HTTPヘッダインジェクション**は、ユーザーが入力したデータが適切にサニタイズされずにHTTPヘッダーに組み込まれる場合に発生する攻撃です。この攻撃は、CRLFインジェクションを含む広範なカテゴリの攻撃を指します。具体的には、HTTPヘッダー全体に対するインジェクション攻撃を指し、攻撃者は新しいヘッダーの追加や、既存のヘッダーの改ざん、さらにはレスポンス全体の操作を行うことができます。

### **違い**

- **焦点**:
  - **CRLFインジェクション**は、主にCRLF文字を利用してヘッダーの終了と新しいヘッダーやボディの開始を操作する攻撃に焦点を当てています。
  - **HTTPヘッダインジェクション**は、CRLFインジェクションを含む、HTTPヘッダー全体に対する任意の入力の注入や改ざんを狙った攻撃を指す、より広い概念です。

- **影響範囲**:
  - **CRLFインジェクション**は、特にヘッダーとボディの境界を操作することで、追加のヘッダーを挿入したり、レスポンスのボディ部分に不正なデータを注入したりすることにフォーカスしています。
  - **HTTPヘッダインジェクション**は、特定のヘッダーだけでなく、複数のヘッダーや全体のレスポンス構造を対象とし、より多様な攻撃が可能です。

### **まとめ**

- **CRLFインジェクション**は、HTTPレスポンスヘッダーにおいて、ヘッダーの終了と新しいヘッダーやボディの開始を操作する攻撃に特化しています。
- **HTTPヘッダインジェクション**は、CRLFインジェクションを含む、広範なHTTPヘッダー操作を可能にする攻撃で、ヘッダー全体の操作やレスポンスの構造を変更することができる攻撃です。

これらの違いを理解し、両方の攻撃手法に対する適切な防御策を講じることが、Webアプリケーションのセキュリティを強化するために重要です。


---
### CRLFインジェクションしないHTTPヘッダインジェクション


**CRLFインジェクションでないHTTPヘッダインジェクションの例**について説明します。

### **HTTPヘッダインジェクションの概要**

HTTPヘッダインジェクションは、攻撃者が不正なデータをHTTPヘッダーに挿入することを狙った攻撃です。HTTPヘッダインジェクションの一部にCRLFインジェクションが含まれますが、ここではCRLFを用いない形のインジェクションの例について説明します。

### **HTTPヘッダインジェクションの例**

#### **PHPの例**

```php
<?php
$user = $_GET['user'];
header("X-User: " . $user);
```

**攻撃手法：**
- 攻撃者が以下のようなリクエストを送信します。

  ```bash
  http://example.com/vulnerable.php?user=John<script>alert('XSS')</script>
  ```

- このリクエストによって、レスポンスヘッダーが以下のように生成されます。

  ```
  X-User: John<script>alert('XSS')</script>
  ```

  このヘッダーがブラウザや他のシステムで表示されると、`<script>`タグが解釈され、XSS攻撃が発生する可能性があります。

#### **Python（Flask）の例**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    user_agent = request.args.get('user_agent')
    response = app.response_class(
        response='Hello, World!',
        status=200,
        headers={'User-Agent': user_agent}
    )
    return response

if __name__ == '__main__':
    app.run()
```

**攻撃手法：**
- 攻撃者が以下のようなURLを使用します。

  ```bash
  http://localhost:5000/?user_agent=Mozilla/5.0<script>alert('XSS')</script>
  ```

- これにより、レスポンスヘッダーに不正なスクリプトが挿入される可能性があります。

  ```
  User-Agent: Mozilla/5.0<script>alert('XSS')</script>
  ```

  ブラウザや解析ツールがこのヘッダーを不適切に解釈した場合、XSS攻撃が成立します。

#### **Node.js（Express）の例**

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    const referrer = req.query.referrer;
    res.setHeader('Referrer', referrer);
    res.send('Hello, World!');
});

app.listen(3000);
```

**攻撃手法：**
- 攻撃者が以下のようなリクエストを送信します。

  ```bash
  http://localhost:3000/?referrer=evilsite.com<script>alert('XSS')</script>
  ```

- これにより、レスポンスヘッダーに不正なスクリプトが挿入される可能性があります。

  ```
  Referrer: evilsite.com<script>alert('XSS')</script>
  ```

#### **Java（Spring）の例**

```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.stereotype.Controller;

import javax.servlet.http.HttpServletResponse;

@Controller
public class HeaderController {

    @GetMapping("/set-header")
    public void setHeader(@RequestParam String value, HttpServletResponse response) {
        response.setHeader("X-Example", value);
    }
}
```

**攻撃手法：**
- 攻撃者が以下のようにリクエストを送信します。

  ```bash
  http://localhost:8080/set-header?value=foo<script>alert('XSS')</script>
  ```

- これにより、以下のようなレスポンスヘッダーが生成されます。

  ```
  X-Example: foo<script>alert('XSS')</script>
  ```

---

### **防止策**

1. **ユーザー入力のサニタイズ**:
   - ヘッダーに挿入されるユーザー入力は、エスケープまたはサニタイズ処理を行い、不正な文字列が含まれないようにします。

   **PHPの例**:
   ```php
   $user = htmlspecialchars($_GET['user'], ENT_QUOTES, 'UTF-8');
   header("X-User: " . $user);
   ```

   **Python（Flask）の例**:
   ```python
   import html
   user_agent = html.escape(request.args.get('user_agent'))
   ```

2. **HTTPヘッダーの構造に適さない文字列を除去**:
   - ヘッダーに挿入されるデータがヘッダーとして無効な文字列を含んでいないかをチェックします。

   **Node.jsの例**:
   ```javascript
   const referrer = req.query.referrer.replace(/[^a-zA-Z0-9\-.]/g, '');
   res.setHeader('Referrer', referrer);
   ```

3. **セキュリティフレームワークの利用**:
   - 可能な限り、ユーザー入力をHTTPヘッダーに直接使用しないようにし、セキュアなライブラリやフレームワークを利用します。

---

**まとめ**:
CRLFインジェクションはHTTPヘッダインジェクションの一種ですが、HTTPヘッダインジェクション全般には、ユーザー入力がエスケープされないままHTTPヘッダーに挿入されることで、不正な動作が発生するリスクがあります。ユーザー入力のエスケープやサニタイズ、無効な文字列の除去などの防止策を実施することで、これらの脆弱性を防ぐことができます。