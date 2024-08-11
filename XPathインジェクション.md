**XPathインジェクション攻撃について**

### **概要**

XPathインジェクション攻撃は、WebアプリケーションがXMLデータベースに対してユーザー入力に基づくXPathクエリを実行する際に発生する脆弱性です。攻撃者は、ユーザー入力に悪意のあるXPathクエリを含めることで、データベースから意図しない情報を取得したり、不正な操作を実行したりすることができます。XPathインジェクションは、SQLインジェクションに似た攻撃手法ですが、対象となるのはSQLデータベースではなく、XMLデータベースやXML文書です。

---

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP**

**脆弱なコード:**
```php
<?php
$xml = simplexml_load_file('users.xml');

$username = $_GET['username'];
$password = $_GET['password'];

$query = "//user[username/text()='$username' and password/text()='$password']";
$result = $xml->xpath($query);

if (!empty($result)) {
    echo "Login successful!";
} else {
    echo "Invalid username or password.";
}
?>
```

**攻撃手法:**
攻撃者が`username`フィールドに`' or '1'='1`を入力し、`password`フィールドに`' or '1'='1`を入力すると、次のようなXPathクエリが生成されます。

```xpath
//user[username/text()='' or '1'='1' and password/text()='' or '1'='1']
```

このクエリは常に`true`を返すため、どのユーザーに対しても認証が成功してしまいます。

#### **Python**

**脆弱なコード:**
```python
import xml.etree.ElementTree as ET

tree = ET.parse('users.xml')
root = tree.getroot()

username = input("Enter username: ")
password = input("Enter password: ")

query = f".//user[username='{username}' and password='{password}']"
result = root.findall(query)

if result:
    print("Login successful!")
else:
    print("Invalid username or password.")
```

**攻撃手法:**
同様に、`username`フィールドに`' or '1'='1`、`password`フィールドに`' or '1'='1`を入力すると、常に`true`を返すクエリが生成され、認証が突破されます。

#### **Node.js**

**脆弱なコード:**
```javascript
const fs = require('fs');
const xpath = require('xpath');
const dom = require('xmldom').DOMParser;

const xml = fs.readFileSync('users.xml', 'utf-8');
const doc = new dom().parseFromString(xml);

const username = req.query.username;
const password = req.query.password;

const query = `//user[username/text()='${username}' and password/text()='${password}']`;
const nodes = xpath.select(query, doc);

if (nodes.length > 0) {
    res.send("Login successful!");
} else {
    res.send("Invalid username or password.");
}
```

**攻撃手法:**
攻撃者は、`username`フィールドに`' or '1'='1`、`password`フィールドに`' or '1'='1`を入力することで、任意のユーザーとして認証を通過できます。

#### **Java**

**脆弱なコード:**
```java
import javax.xml.xpath.*;
import org.w3c.dom.*;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;

public class XPathInjectionExample {
    public static void main(String[] args) throws Exception {
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        DocumentBuilder builder = factory.newDocumentBuilder();
        Document doc = builder.parse("users.xml");

        XPathFactory xPathfactory = XPathFactory.newInstance();
        XPath xpath = xPathfactory.newXPath();

        String username = args[0];
        String password = args[1];

        String expression = "//user[username/text()='" + username + "' and password/text()='" + password + "']";
        XPathExpression expr = xpath.compile(expression);

        NodeList nodes = (NodeList) expr.evaluate(doc, XPathConstants.NODESET);

        if (nodes.getLength() > 0) {
            System.out.println("Login successful!");
        } else {
            System.out.println("Invalid username or password.");
        }
    }
}
```

**攻撃手法:**
攻撃者が`username`として`' or '1'='1`、`password`として`' or '1'='1`を提供することで、XPathクエリが改ざんされ、認証を突破することができます。

---

### **防止策**

1. **パラメータ化されたクエリの使用:**
   - SQLインジェクション防止と同様に、XPathクエリをパラメータ化し、ユーザー入力が直接クエリに挿入されないようにする方法です。

   例えば、Pythonでは以下のように構築します。
   ```python
   query = ".//user[username=? and password=?]"
   result = root.findall(query, {'username': username, 'password': password})
   ```

2. **ユーザー入力のエスケープとサニタイズ:**
   - ユーザー入力に含まれる特殊文字（`'`, `"`, `&`, `<`, `>`, など）をエスケープし、意図しないクエリの改ざんを防ぎます。
   - 例えば、ユーザー入力をXMLに安全な文字列に変換します。

3. **ホワイトリスト方式の導入:**
   - ユーザー入力が特定のフォーマット（例えば、アルファベットや数字のみ）に制限される場合、その制限を適用し、危険な入力が行われないようにします。

4. **エラーメッセージの管理:**
   - エラーメッセージにクエリ構造やデータベースの詳細が含まれないようにし、攻撃者に情報を提供しないようにします。

5. **セキュリティライブラリの使用:**
   - 安全なXPathクエリを作成するためのライブラリやフレームワークを使用し、可能な限り既知のセキュリティ問題に対処することが重要です。

---

**まとめ:**
XPathインジェクション攻撃は、XMLデータベースや文書に対して不正なクエリを実行するための脆弱性です。パラメータ化されたクエリの使用、入力のサニタイズ、ホワイトリスト方式の導入などの防止策を実装することで、この脆弱性を効果的に防ぐことが可能です。どのプラットフォームや言語でも、これらのベストプラクティスを適用することが推奨されます。