---
title: ReDoS
---

### **概要**

ReDoS（Regular Expression Denial of Service）攻撃は、正規表現の処理において計算量が急激に増加するケースを悪用して、システムのリソースを過剰に消費させることで、サービス拒否（DoS）状態を引き起こす攻撃です。特に、バックトラッキングが多発する複雑な正規表現や、非効率的な正規表現を使用している場合に、この攻撃が可能になります。

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP**

**脆弱なコード:**
```php
<?php
$input = $_GET['input'];
if (preg_match('/^(a+)+$/', $input)) {
    echo "Match found!";
} else {
    echo "No match.";
}
?>
```

**攻撃手法:**
このコードは、`a`の連続する文字列をチェックするために正規表現を使用していますが、入力として大量の`a`と無関係な文字を含む文字列（例: `aaaaaaaaaaaaaaaaaaaaX`）を送信することで、正規表現の処理が長時間にわたり続き、サーバーが応答しなくなる可能性があります。

#### **Python**

**脆弱なコード:**
```python
import re

input_string = input("Enter a string: ")
pattern = re.compile(r'^(a+)+$')

if pattern.match(input_string):
    print("Match found!")
else:
    print("No match.")
```

**攻撃手法:**
Pythonでも同様に、`aaaaaaaaaaaaaaaaaaaaX`のような入力を与えることで、正規表現の処理時間が劇的に増加し、システムリソースを大量に消費します。

#### **Node.js**

**脆弱なコード:**
```javascript
const input = req.query.input;
const regex = /^(a+)+$/;

if (regex.test(input)) {
    res.send("Match found!");
} else {
    res.send("No match.");
}
```

**攻撃手法:**
Node.jsでも、複雑な正規表現に対して攻撃者が巧妙に構成された入力を送信することで、CPU使用率が高まり、サービスが停止する可能性があります。

#### **Java**

**脆弱なコード:**
```java
import java.util.regex.*;

public class ReDoSExample {
    public static void main(String[] args) {
        String input = args[0];
        Pattern pattern = Pattern.compile("^(a+)+$");
        
        Matcher matcher = pattern.matcher(input);
        if (matcher.find()) {
            System.out.println("Match found!");
        } else {
            System.out.println("No match.");
        }
    }
}
```

**攻撃手法:**
Javaでも、複雑な正規表現に対して攻撃者が膨大な入力（例: `aaaaaaaaaaaaaaaaaaaaX`）を提供することで、正規表現の評価が非常に遅くなり、システムが応答しなくなる可能性があります。

---

### **防止策**

1. **正規表現の最適化:**
   - 脆弱な正規表現を使用しないように、可能な限りシンプルかつ効率的な正規表現を作成します。例えば、バックトラッキングが多発するパターンを避け、代替のアプローチを検討します。
   - 例: `^(a+)+$`の代わりに、`^a+$`のようなよりシンプルな正規表現を使用する。

2. **入力の制限:**
   - ユーザーからの入力に対してサイズや形式の制限を設け、悪意のある入力が長時間処理されることを防ぎます。例えば、入力の最大長を制限することが有効です。

3. **正規表現のタイムアウト設定:**
   - 一部の言語やフレームワークでは、正規表現の処理にタイムアウトを設定できる場合があります。これにより、処理が一定時間を超えると強制的に中断させることができます。
   - 例: Javaでは`java.util.regex.Matcher`で正規表現のタイムアウトを実装することが可能です。

4. **正規表現の事前検証:**
   - 使用する正規表現を事前に検証し、パフォーマンス上の問題がないかチェックします。これにより、問題が発生する可能性のある正規表現を未然に排除できます。

5. **ライブラリのアップデートと脆弱性管理:**
   - 使用している言語やフレームワークの正規表現処理に関する脆弱性が報告された場合、直ちにパッチを適用し、アップデートを行うことが重要です。

---

**まとめ:**
ReDoS攻撃は、正規表現の非効率的な処理を悪用することでサービス拒否状態を引き起こす攻撃です。複雑な正規表現を最適化することや、入力制限、タイムアウト設定、ライブラリのアップデートを通じて、この脆弱性を効果的に防ぐことが可能です。各言語に共通する問題であるため、特に注意を払い、対策を講じることが求められます。