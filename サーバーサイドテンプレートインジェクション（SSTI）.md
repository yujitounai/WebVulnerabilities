---
title: サーバーサイドテンプレートインジェクション
---

**SSTI（Server-Side Template Injection）攻撃について**

### **概要**

SSTI（Server-Side Template Injection）攻撃は、Webアプリケーションがテンプレートエンジンを使用して動的なコンテンツを生成する際に、ユーザーからの入力を適切に処理せずにテンプレート内で評価してしまうことによって発生する脆弱性です。攻撃者は、この脆弱性を利用して、テンプレートエンジン内で任意のコードを実行し、サーバーの制御を乗っ取ったり、機密データにアクセスしたりすることができます。

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP**

**脆弱なコード:**
```php
<?php
$template = $_GET['template'];
echo eval("?> $template <?php ");
?>
```

**攻撃手法:**
攻撃者が`template`パラメータに`<?= system('whoami'); ?>`のようなコードを挿入すると、そのコードがサーバー上で実行され、`whoami`コマンドの結果が表示されます。

#### **Python（Flask + Jinja2）**

**脆弱なコード:**
```python
from flask import Flask, request, render_template_string

app = Flask(__name__)

@app.route('/')
def index():
    template = request.args.get('template')
    return render_template_string(template)

if __name__ == '__main__':
    app.run()
```

**攻撃手法:**
攻撃者は、`template`パラメータに`{{ 7*7 }}`のようなコードを送信すると、Jinja2テンプレートエンジンによって計算され、`49`という結果が表示されます。さらに、`{{ config.items() }}`のようなコードを利用して、アプリケーションの設定情報を漏洩させることも可能です。

#### **Node.js（Express + Pug）**

**脆弱なコード:**
```javascript
const express = require('express');
const app = express();
const pug = require('pug');

app.get('/', (req, res) => {
    const template = req.query.template;
    const compiledFunction = pug.compile(template);
    res.send(compiledFunction());
});

app.listen(3000);
```

**攻撃手法:**
攻撃者は、`template`パラメータに`${process.env}`のようなコードを挿入して送信すると、環境変数の内容が表示される可能性があります。また、`process.exit()`などのコードを使用して、サーバーを停止させることも可能です。

#### **Java（Thymeleaf）**

**脆弱なコード:**
```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.stereotype.Controller;
import org.thymeleaf.context.Context;
import org.thymeleaf.TemplateEngine;

@Controller
public class SSTIController {

    private final TemplateEngine templateEngine;

    public SSTIController(TemplateEngine templateEngine) {
        this.templateEngine = templateEngine;
    }

    @GetMapping("/")
    public String index(@RequestParam String template) {
        Context context = new Context();
        return templateEngine.process(template, context);
    }
}
```

**攻撃手法:**
Thymeleafテンプレートエンジンでは、`template`パラメータに`[[${T(java.lang.Runtime).getRuntime().exec('whoami')}`のようなコードを挿入することで、任意のシステムコマンドを実行できる可能性があります。

---

### **防止策**

1. **ユーザー入力の検証とサニタイズ:**
   - ユーザーからの入力をテンプレートエンジンに渡す前に、必ずサニタイズとバリデーションを行います。特に、テンプレートに直接挿入されるユーザー入力には、特殊な文字やコードが含まれていないことを確認します。

2. **テンプレートエンジンの安全な使用:**
   - テンプレートエンジンの使用において、ユーザーが直接テンプレートコードを操作できないようにする必要があります。可能であれば、テンプレートコードとデータの結合をサーバーサイドで安全に処理します。

3. **コンテンツセキュリティポリシー（CSP）の設定:**
   - コンテンツセキュリティポリシー（CSP）を設定し、サーバーからの信頼できるスクリプトのみを許可することで、クライアントサイドでの攻撃を防ぐことができます。

4. **安全なテンプレートエンジンの選択:**
   - サンドボックス機能を持つテンプレートエンジンを選択し、任意のコード実行ができないように制限されたテンプレートエンジンを使用します。

5. **コードレビューとテスト:**
   - テンプレートエンジンを使用するコード部分を重点的にコードレビューし、SSTIの脆弱性がないか確認します。また、セキュリティテストを行い、実際にSSTI攻撃が発生しないことを確認します。

---

**まとめ:**
SSTI攻撃は、テンプレートエンジンを使用してユーザー入力を処理する際に発生する可能性がある脆弱性です。ユーザー入力の厳格なサニタイズ、テンプレートエンジンの適切な使用、コンテンツセキュリティポリシーの設定などを通じて、この脆弱性を防ぐことが可能です。どの言語やフレームワークでも、これらの防止策を適用することが推奨されます。