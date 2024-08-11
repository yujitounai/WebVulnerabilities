**XML外部実体攻撃（XXE）について**

### **概要**

XXE（XML External Entity）攻撃は、XMLパーサーが外部エンティティの解決を許可している場合に発生する脆弱性です。この攻撃により、攻撃者はXMLファイル内で定義された外部エンティティを利用して、サーバー上の機密ファイルの読み取り、サービス拒否（DoS）、内部ネットワークへのアクセスなどを行うことができます。XXEは、特にXMLを処理するアプリケーションにおいて、外部エンティティの処理が不適切に行われている場合に発生します。

---

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP**

**脆弱なコード:**
```php
<?php
$xmlString = file_get_contents('php://input');
$xml = simplexml_load_string($xmlString);
echo $xml->asXML();
?>
```

**攻撃手法:**
攻撃者は以下のような悪意のあるXMLを送信することで、サーバー上の任意のファイルを読み取ることができます。

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

この攻撃により、サーバーの`/etc/passwd`ファイルの内容が読み取られ、出力されます。

#### **Python**

**脆弱なコード:**
```python
import xml.etree.ElementTree as ET

xml_string = input("Enter XML data: ")
root = ET.fromstring(xml_string)
print(ET.tostring(root))
```

**攻撃手法:**
同様に、以下の悪意のあるXMLを入力すると、サーバー上の任意のファイルが読み取られる可能性があります。

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

#### **Node.js**

**脆弱なコード:**
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const xml2js = require('xml2js');

const app = express();
app.use(bodyParser.text({ type: 'application/xml' }));

app.post('/xml', (req, res) => {
    xml2js.parseString(req.body, (err, result) => {
        res.send(result);
    });
});

app.listen(3000);
```

**攻撃手法:**
攻撃者が以下のような悪意のあるXMLを送信すると、サーバー上の機密ファイルが読み取られる可能性があります。

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

#### **Java**

**脆弱なコード:**
```java
import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;
import org.w3c.dom.Document;

public class XXEExample {
    public static void main(String[] args) throws Exception {
        DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
        DocumentBuilder builder = factory.newDocumentBuilder();
        Document doc = builder.parse("input.xml");
        System.out.println(doc.getDocumentElement().getTextContent());
    }
}
```

**攻撃手法:**
攻撃者が悪意のあるXMLファイルを提供すると、サーバー上の機密ファイルが読み取られる可能性があります。

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
<root>&xxe;</root>
```

---

### **防止策**

1. **外部エンティティの処理を無効化する:**
   - XMLパーサーで外部エンティティの処理を無効にすることで、XXE攻撃を防止します。

   - **PHP**: `libxml_disable_entity_loader(true);`を使用。
   - **Python**: `defusedxml`モジュールを使用して、安全なXML処理を行います。
   - **Node.js**: `xml2js`などのライブラリで外部エンティティの処理を無効にします。
   - **Java**:
     ```java
     DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
     factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
     factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
     factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
     ```

2. **信頼できるデータソースのみを処理する:**
   - 外部からの入力としてXMLを受け取る場合、そのデータソースが信頼できるかどうかを確認し、不正なXMLデータの処理を避けます。

3. **XMLパーサーの設定を安全にする:**
   - XXE攻撃に対する脆弱性がないように、XMLパーサーの設定を確認し、安全な設定を行います。

4. **入力データの検証とサニタイズ:**
   - XMLデータの入力を受け取る前に、その内容をサニタイズし、潜在的に危険な内容が含まれていないか検証します。

5. **使用するXMLパーサーやライブラリのアップデート:**
   - XMLパーサーやライブラリを常に最新の状態に保ち、セキュリティパッチを適用します。

---

**まとめ:**
XXE攻撃は、XMLパーサーが外部エンティティの解決を許可している場合に発生する脆弱性です。外部エンティティの処理を無効にすること、信頼できるデータソースのみを処理すること、パーサーの安全な設定を行うことなどが防止策として有効です。各言語での適切な設定とベストプラクティスを適用することで、この脆弱性を効果的に防ぐことが可能です。


### **PHPにおけるXXE攻撃の制限について**

XXE（XML External Entity）攻撃は、XMLパーサーが外部エンティティの解決を許可している場合に発生する脆弱性ですが、PHPの新しいバージョンでは、この攻撃が限定的にしか行えないように改善されています。

#### **PHPのバージョンによるXXE対策**

PHPのバージョン7.0以降では、デフォルトで`libxml_disable_entity_loader()`が`true`に設定されており、外部エンティティの読み込みが無効化されています。これにより、XMLパーサーで外部エンティティを自動的に解決することが防止され、XXE攻撃のリスクが大幅に軽減されています。

**具体的な対策内容:**
- **libxml_disable_entity_loader() のデフォルト設定**: PHP 7.0以降では、この関数が自動的に外部エンティティの読み込みを無効にするため、攻撃者が外部リソースにアクセスすることが難しくなります。
- **ファイルシステムベースのエンティティの制限**: `file://`スキームなどを使ってローカルファイルにアクセスするエンティティは、デフォルトでブロックされます。
  
#### **PHP 7.0以降でのXXE攻撃の影響**

以下の点で、PHP 7.0以降でのXXE攻撃は限定的です。

1. **外部エンティティの読み込みが無効化**:
   - 外部エンティティ（`SYSTEM`や`PUBLIC`エンティティなど）を使用した、リモートリソースやローカルファイルへのアクセスが原則として無効化されています。
   - これにより、攻撃者は`file://`スキームを使って任意のファイルを読み込むといった典型的なXXE攻撃を仕掛けることが難しくなっています。

2. **既存のXMLファイルへのアクセス**:
   - 攻撃者が任意のXMLファイルを生成したり、既存のXMLファイルを利用して不正な操作を行うことは依然として可能ですが、リモートリソースへのアクセスやファイルシステムからのデータの引き出しが難しくなっています。

#### **限定的な攻撃の可能性**

新しいバージョンのPHPでは、完全にXXE攻撃が防止されたわけではありませんが、以下のような限定的な攻撃が残されています。

- **内部エンティティの悪用**: 
  内部エンティティを用いて、同じXML文書内のデータを繰り返し使用することは可能です。ただし、これは外部リソースへのアクセスを必要としないため、攻撃の影響は非常に限定的です。
  
- **DoS攻撃**: 
  巨大なエンティティを使用することで、サーバーのリソースを消費させ、サービス拒否（DoS）状態を引き起こすことが可能です。しかし、この手法も効果的な攻撃とは言い難いです。

#### **PHPでの安全なXML処理**

**PHP 7.0以降での標準設定**であれば、外部エンティティの無効化は自動的に適用されますが、追加の安全対策として以下の設定を確認してください。

```php
libxml_disable_entity_loader(true); // 外部エンティティの読み込みを無効化
$xml = simplexml_load_string($xmlString, 'SimpleXMLElement', LIBXML_NOENT | LIBXML_DTDLOAD | LIBXML_DTDATTR); // DTDやエンティティの読み込みを無効化
```

#### **まとめ**

PHPの新しいバージョンでは、XXE攻撃は限定的にしかできないように設計されています。特に外部エンティティの読み込みがデフォルトで無効化されているため、リモートファイルの読み取りやサーバーファイルへのアクセスなど、典型的なXXE攻撃のリスクは大幅に減少しています。しかしながら、完全な安全性を確保するために、外部エンティティの無効化を確認し、XMLパーサーの安全な設定を徹底することが重要です。

他の主要な言語でも、XXE（XML External Entity）攻撃に対する対策が施されています。各言語の標準ライブラリやフレームワークには、デフォルトでXXE攻撃を防ぐための設定が含まれている場合があり、最新のバージョンではセキュリティが強化されています。

### **PythonにおけるXXE攻撃の制限について**

#### **デフォルトの対策**
- **Python 3.3以降**の`xml.etree.ElementTree`や`xml.dom.minidom`では、外部エンティティの解決はデフォルトで無効化されています。ただし、これらのモジュールは完全に安全とは限らないため、セキュリティを強化するために専用の対策が推奨されます。

#### **推奨される安全なライブラリ**
- **`defusedxml`モジュール**: Pythonでは、`defusedxml`というモジュールを使用することで、安全にXMLを処理できます。このモジュールは、外部エンティティの処理を自動的に無効化し、XXE攻撃を防ぎます。

   ```python
   import defusedxml.ElementTree as ET

   tree = ET.parse('input.xml')
   root = tree.getroot()
   ```

### **Node.jsにおけるXXE攻撃の制限について**

#### **デフォルトの対策**
- Node.jsの標準ライブラリ`xml2js`や`xmldom`では、外部エンティティの解決がデフォルトで有効な場合があります。このため、手動で外部エンティティの無効化を行う必要があります。

#### **推奨される安全な設定**
- **`xmldom`モジュールの使用例**:
   ```javascript
   const { DOMParser } = require('xmldom');

   const parser = new DOMParser({
       errorHandler: { warning: null, error: null }, // エラーハンドリングを設定
       entityMap: {} // 外部エンティティのマッピングを空に設定
   });

   const doc = parser.parseFromString('<xml>...</xml>', 'text/xml');
   ```

### **JavaにおけるXXE攻撃の制限について**

#### **デフォルトの対策**
- JavaのXMLパーサー（JAXP、DOM、SAXなど）では、過去のバージョンでは外部エンティティの解決がデフォルトで有効でしたが、**Java 8以降**ではデフォルトで無効化されています。ただし、古いバージョンや特定の設定では無効化されていない可能性があるため、明示的に設定することが推奨されます。

#### **推奨される安全な設定**
- **DOMパーサーの場合**:
   ```java
   DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
   factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
   factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
   factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);

   DocumentBuilder builder = factory.newDocumentBuilder();
   Document doc = builder.parse("input.xml");
   ```

- **SAXパーサーの場合**:
   ```java
   SAXParserFactory factory = SAXParserFactory.newInstance();
   factory.setFeature("http://xml.org/sax/features/external-general-entities", false);
   factory.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
   factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);

   SAXParser parser = factory.newSAXParser();
   parser.parse("input.xml", new DefaultHandler());
   ```

### **その他の言語**

- **C# (.NET Framework/.NET Core)**:
  - **.NET Framework 4.5.2**以降、または**.NET Core**では、XML処理の際に外部エンティティが無効化されています。ただし、古いバージョンでは設定が必要です。

  ```csharp
  var settings = new XmlReaderSettings();
  settings.DtdProcessing = DtdProcessing.Prohibit; // DTD処理を禁止
  settings.XmlResolver = null; // 外部エンティティの解決を無効化

  using (var reader = XmlReader.Create("input.xml", settings)) {
      var doc = new XmlDocument();
      doc.Load(reader);
  }
  ```

- **Ruby**:
  - **REXML**や**Nokogiri**を使用している場合、外部エンティティの解決はデフォルトで無効化されていますが、念のため設定を確認することが推奨されます。

  ```ruby
  require 'nokogiri'

  xml = Nokogiri::XML(File.read('input.xml')) do |config|
      config.nonet.noent
  end
  ```

### **まとめ**

多くのプログラミング言語では、XXE攻撃に対する対策がデフォルトで組み込まれているか、設定によって容易に防御できるようになっています。しかし、古いバージョンや特定の設定では依然として脆弱性が残る場合があるため、開発環境や使用しているライブラリが最新であることを確認し、必要に応じて追加のセキュリティ設定を行うことが重要です。