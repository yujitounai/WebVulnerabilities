---
title: SSIインジェクション
---


**SSIインジェクション攻撃について**

### **概要**

SSI（Server-Side Includes）インジェクションは、Webアプリケーションがユーザーからの入力を元にサーバーサイドのインクルード機能を使用して動的にコンテンツを生成する際に発生する脆弱性です。攻撃者は、この脆弱性を利用して、サーバー上で任意のコードを実行させたり、機密情報にアクセスしたりすることができます。SSIは通常、静的HTMLページにサーバーサイドで動的な要素を組み込むために使用されますが、これが不適切に管理されると、深刻なセキュリティリスクとなります。

---

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP**

**脆弱なコード:**
```php
<?php
$filename = $_GET['filename'];
include($filename);
?>
```

**攻撃手法:**
攻撃者が`filename`に悪意のあるSSIコードを挿入すると、例えば次のようにサーバーで任意のコマンドを実行させることができます。
```bash
http://example.com/page.php?filename=|whoami|
```
これにより、`whoami`コマンドがサーバー上で実行され、サーバーのユーザー名が表示されます。

#### **Python**

**脆弱なコード:**
```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route('/')
def index():
    template = request.args.get('template')
    return render_template_string(template)
```

**攻撃手法:**
攻撃者は、`template`パラメータに悪意のあるSSIコードを挿入して、例えば次のような攻撃を行うことができます。
```bash
http://example.com/?template=<ssi><!--#exec cmd="ls" --><!--#echo var="DOCUMENT_ROOT"-->
```
これにより、サーバー上のディレクトリリストや環境変数が攻撃者に表示されます。

#### **Node.js**

**脆弱なコード:**
```javascript
const express = require('express');
const app = express();
const fs = require('fs');

app.get('/', (req, res) => {
    const filename = req.query.filename;
    const fileContent = fs.readFileSync(filename, 'utf8');
    res.send(fileContent);
});

app.listen(3000);
```

**攻撃手法:**
攻撃者は`filename`パラメータに悪意のあるSSIコードを指定し、例えば次のように任意のファイル内容を取得できます。
```bash
http://example.com/?filename=|/etc/passwd|
```
これにより、サーバーの`/etc/passwd`ファイルの内容が表示される可能性があります。

#### **Java**

**脆弱なコード:**
```java
import javax.servlet.*;
import javax.servlet.http.*;
import java.io.*;

public class SSIExampleServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String filename = request.getParameter("filename");
        RequestDispatcher dispatcher = request.getRequestDispatcher(filename);
        dispatcher.include(request, response);
    }
}
```

**攻撃手法:**
攻撃者は、`filename`パラメータに悪意のあるファイルパスを指定し、次のようにしてサーバーの内部ファイルを表示させることができます。
```bash
http://example.com/SSIExampleServlet?filename=/WEB-INF/web.xml
```
これにより、サーバーの`WEB-INF/web.xml`ファイルが攻撃者に送信されます。

#### **Perl**

**脆弱なコード:**
```perl
#!/usr/bin/perl

my $filename = $ARGV[0];
open(my $fh, '<', $filename) or die "Cannot open file: $!";
while (my $line = <$fh>) {
    print $line;
}
close($fh);
```

**攻撃手法:**
攻撃者は`filename`パラメータに悪意のあるSSIコードを挿入して、例えば次のような攻撃を行うことができます。
```bash
perl script.pl "| ls |"
```
これにより、サーバー上で`ls`コマンドが実行され、その結果が出力されます。

---

### **防止策**

1. **入力のサニタイズとバリデーション:**
   - ユーザーからの入力をサニタイズし、SSIコードやその他の特殊文字を無効化します。例えば、特殊文字や危険なコマンドのフィルタリングを行います。

2. **SSIの無効化:**
   - Webサーバーの設定でSSIを無効化します。SSIが不要な場合は、サーバーでこれを無効にすることでリスクを低減できます。

3. **ホワイトリスト方式の導入:**
   - インクルードするファイルやコマンドをホワイトリストに登録し、許可されたもの以外は実行しないようにします。

4. **コンテンツセキュリティポリシー（CSP）の使用:**
   - コンテンツセキュリティポリシーを設定して、信頼できるソースからのスクリプトのみを許可することで、SSIやXSSのリスクを軽減します。

5. **アクセス制御の強化:**
   - サーバーやアプリケーションのアクセス制御を強化し、攻撃者が重要なファイルやディレクトリにアクセスできないようにします。

---

**まとめ:**
SSIインジェクションは、サーバーサイドインクルード機能を悪用して任意のコードを実行する攻撃です。入力のサニタイズ、SSIの無効化、ホワイトリスト方式の導入などの防止策を適用することで、この脆弱性を効果的に防ぐことができます。どの言語やフレームワークでも、これらのベストプラクティスを実装することが推奨されます。