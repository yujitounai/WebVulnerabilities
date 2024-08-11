### JWT（JSON Web Token）に対する攻撃

#### 概要

JWT（JSON Web Token）は、情報を安全に伝達するために使用されるコンパクトでURLセーフなトークン形式です。一般的には、ユーザー認証やAPIの認可に使用されます。JWTには3つの部分、すなわちヘッダー、ペイロード、署名が含まれています。署名はトークンの改ざんを防ぐためのものですが、適切に実装されていない場合、攻撃者に悪用される可能性があります。

JWTに対する攻撃は、主に以下の手法に分類されます。

1. **アルゴリズムの変更攻撃**
2. **署名のないトークンの使用**
3. **キーの推測**
4. **ペイロードの不正操作**

### 脆弱性を持つコードの例と攻撃手法

各言語ごとにJWTに対する攻撃がどのように実行されるかを説明します。基本的な攻撃手法は言語に依存しないため、共通の原理を基に解説します。

#### 1. アルゴリズムの変更攻撃

**脆弱性を持つコードの例:**

- **PHP (Firebase JWTライブラリの例)**

```php
use Firebase\JWT\JWT;

$jwt = $_GET['token'];  // 外部からの入力としてトークンを取得

$key = 'your-256-bit-secret';
$decoded = JWT::decode($jwt, $key, array('HS256'));

echo json_encode($decoded);
```

- **Python (PyJWTライブラリの例)**

```python
import jwt

token = request.args.get('token')  # 外部からの入力としてトークンを取得
secret = 'your-256-bit-secret'

try:
    decoded = jwt.decode(token, secret, algorithms=['HS256'])
    return jsonify(decoded)
except jwt.DecodeError:
    return 'Invalid token', 403
```

- **Node.js (jsonwebtokenライブラリの例)**

```javascript
const jwt = require('jsonwebtoken');

const token = req.query.token;  // 外部からの入力としてトークンを取得
const secret = 'your-256-bit-secret';

try {
    const decoded = jwt.verify(token, secret, { algorithms: ['HS256'] });
    res.json(decoded);
} catch (err) {
    res.status(403).send('Invalid token');
}
```

- **Java (JJWTライブラリの例)**

```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

String token = request.getParameter("token");  // 外部からの入力としてトークンを取得
String secretKey = "your-256-bit-secret";

Claims claims = Jwts.parser()
    .setSigningKey(secretKey.getBytes())
    .parseClaimsJws(token)
    .getBody();

response.getWriter().write(claims.toString());
```

**攻撃手法:**

攻撃者は、JWTのヘッダー部分を変更して署名アルゴリズムを`HS256`から`none`に設定します。これにより、署名が検証されずにトークンが有効とされてしまう可能性があります。以下に攻撃の手順を示します。

1. トークンのヘッダーを`{"alg": "none", "typ": "JWT"}`に変更。
2. 署名部分を削除してトークンを再生成。
3. サーバーに改ざんしたトークンを送信。

この脆弱性が存在すると、サーバーは署名を確認せずにペイロードを信頼するため、攻撃者が任意のユーザーとして認証を通過することが可能になります。

#### 2. 署名のないトークンの使用

**脆弱性を持つコードの例:**

- **PHP**

```php
use Firebase\JWT\JWT;

$jwt = $_GET['token'];  // 外部からの入力としてトークンを取得

$decoded = JWT::decode($jwt, null, array('none'));

echo json_encode($decoded);
```

- **Python**

```python
import jwt

token = request.args.get('token')  # 外部からの入力としてトークンを取得

decoded = jwt.decode(token, options={"verify_signature": False})
return jsonify(decoded)
```

- **Node.js**

```javascript
const jwt = require('jsonwebtoken');

const token = req.query.token;  // 外部からの入力としてトークンを取得

const decoded = jwt.decode(token, { complete: true });
res.json(decoded.payload);
```

- **Java**

```java
import io.jsonwebtoken.Jwts;

String token = request.getParameter("token");  // 外部からの入力としてトークンを取得

Claims claims = Jwts.parser()
    .parseClaimsJws(token)
    .getBody();

response.getWriter().write(claims.toString());
```

**攻撃手法:**

攻撃者は、署名のないトークンをサーバーに送信します。このトークンは、適切に署名が検証されないため、サーバーがそのまま受け入れてしまう場合があります。これにより、攻撃者は任意のペイロードをサーバーに渡すことができ、例えば、権限を昇格させたり、別のユーザーとして操作を行ったりすることが可能です。

#### 3. キーの推測

**脆弱性を持つコードの例:**

- **PHP**

```php
use Firebase\JWT\JWT;

$jwt = $_GET['token'];  // 外部からの入力としてトークンを取得

$key = 'simplekey';  // 脆弱なキー
$decoded = JWT::decode($jwt, $key, array('HS256'));

echo json_encode($decoded);
```

- **Python**

```python
import jwt

token = request.args.get('token')  # 外部からの入力としてトークンを取得
secret = 'simplekey'  # 脆弱なキー

decoded = jwt.decode(token, secret, algorithms=['HS256'])
return jsonify(decoded)
```

- **Node.js**

```javascript
const jwt = require('jsonwebtoken');

const token = req.query.token;  // 外部からの入力としてトークンを取得
const secret = 'simplekey';  // 脆弱なキー

try {
    const decoded = jwt.verify(token, secret, { algorithms: ['HS256'] });
    res.json(decoded);
} catch (err) {
    res.status(403).send('Invalid token');
}
```

- **Java**

```java
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

String token = request.getParameter("token");  // 外部からの入力としてトークンを取得
String secretKey = "simplekey";  // 脆弱なキー

Claims claims = Jwts.parser()
    .setSigningKey(secretKey.getBytes())
    .parseClaimsJws(token)
    .getBody();

response.getWriter().write(claims.toString());
```

**攻撃手法:**

攻撃者はブルートフォース攻撃を用いて、トークンの署名に使用されたシークレットキーを推測します。簡単すぎるシークレットキーを使用している場合、推測が容易であり、攻撃者がトークンを偽造してサーバーに送り込むことができます。これにより、攻撃者は任意のユーザーとしての操作を行ったり、セッションを乗っ取ることが可能です。

### 防止策

1. **強力なシークレットキーを使用する**:
   - ブルートフォース攻撃に耐えるために、ランダムで長い（少なくとも256ビット）シークレットキーを使用します。

2. **アルゴリズムの固定**:
   - トークンの検証時にアルゴリズムを`none`などの危険なものに変更できないように、使用するアルゴリズムを厳密に指定します。

   **例**: `JWT::decode($jwt, $key, array('HS256'));` のように特定のアルゴリズムを指定する。

3. **署名のないトークンを拒否する**:
   - 署名のないトークンを受け入れないようにするため、`none`アルゴリズムや署名検証をオプションにしない。

4. **トークンの有効期限を短く設定する**:
   - トークンが長期間有効であると、その間に攻撃されるリスクが増えるため、短い有効期限を設定し、必要に応じてリフレッシュトークンを使用します。

5. **JWTの内容の検証**:
   - トークンのペイロードに含まれる情報を信頼する前に、その内容を慎

重に検証します。例えば、`sub`（ユーザーID）や`role`（ユーザーの役割）などのフィールドが適切であるか確認します。

6. **セキュリティフレームワークの利用**:
   - JWTの処理を手動で行うのではなく、セキュリティに配慮されたライブラリやフレームワークを使用し、それらを最新の状態に保つようにします。

### まとめ

JWTに対する攻撃は、署名アルゴリズムの不正利用やシークレットキーの弱さを悪用するものが多いです。これらの攻撃を防ぐためには、強力なシークレットキーの使用、アルゴリズムの固定、署名なしトークンの拒否、そしてトークンの内容と有効期限の適切な管理が重要です。各言語で利用可能なセキュアなライブラリを活用し、これらの防止策を実装することが求められます。