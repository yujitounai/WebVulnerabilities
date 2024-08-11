---
title: OSコマンドインジェクション
---

#### OSコマンドインジェクションの概要

OSコマンドインジェクションは、アプリケーションがユーザーからの入力を直接OSのシェルに渡して実行する場合に発生する脆弱性です。攻撃者は、ユーザー入力に悪意のあるコマンドを追加することで、アプリケーションが意図しないコマンドを実行させることができます。この脆弱性を利用することで、攻撃者はシステム上の任意のコマンドを実行し、データの漏洩、システムの破壊、不正アクセスなどを引き起こす可能性があります。

---

### 脆弱性を持つコードの例と攻撃手法

#### **PHP**
**脆弱なコード:**
```php
<?php
$filename = $_GET['filename'];
system("ls " . $filename);
?>
```

**攻撃手法:**
ユーザーが`filename`パラメータに`"; rm -rf /"`のような入力を提供した場合、以下のように解釈されます。
```bash
ls ; rm -rf /
```
これにより、攻撃者はシステム上の重要なファイルを削除することが可能になります。

#### **Python**
**脆弱なコード:**
```python
import os

filename = input("Enter the filename: ")
os.system("ls " + filename)
```

**攻撃手法:**
同様に、`filename`に`"; rm -rf /"`を入力することで、同様の攻撃が可能です。

#### **Node.js**
**脆弱なコード:**
```javascript
const { exec } = require('child_process');

const filename = req.query.filename;
exec('ls ' + filename, (error, stdout, stderr) => {
  console.log(stdout);
});
```

**攻撃手法:**
`filename`に悪意のあるコマンドを挿入することで、同様に任意のコマンドが実行されます。

#### **Java**
**脆弱なコード:**
```java
import java.io.*;

public class CommandInjectionExample {
    public static void main(String[] args) {
        String filename = args[0];
        Runtime rt = Runtime.getRuntime();
        try {
            Process proc = rt.exec("ls " + filename);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**攻撃手法:**
同様に、悪意のある入力によって任意のコマンドが実行される可能性があります。

#### **Perl**
**脆弱なコード:**
```perl
#!/usr/bin/perl

my $filename = $ARGV[0];
system("ls $filename");
```

**攻撃手法:**
`filename`に`"; rm -rf /"`を挿入することで、攻撃が実行されます。

---

### **防止策**

OSコマンドインジェクションを防ぐための基本的な対策は以下の通りです。

1. **入力のサニタイズ**  
   ユーザーからの入力をそのままシェルに渡さないように、入力をエスケープするか、サニタイズする必要があります。

   - PHP: `escapeshellcmd()`や`escapeshellarg()`を使用。
   - Python: `shlex.quote()`を使用。
   - Node.js: `child_process`の`execFile()`メソッドを使用。
   - Java: `ProcessBuilder`を使用。
   - Perl: シェルでなく、Perlのビルトイン関数を使用する。

2. **パラメータ化されたコマンドの使用**  
   コマンドの実行時にパラメータ化された入力を利用することで、入力がコマンドラインとして解釈されることを防ぎます。

3. **シェルコマンドの直接使用を避ける**  
   可能であれば、シェルコマンドの実行自体を避け、システムのAPIやビルトインの関数を使用します。

4. **最小権限の原則**  
   コマンドを実行するプロセスやユーザーが持つ権限を最小限に抑えることで、仮にコマンドインジェクションが発生しても影響を限定することができます。

各言語における防止策は類似していますが、特定の関数やメソッドを使用することで、安全性を向上させることができます。これらの対策を講じることで、OSコマンドインジェクションのリスクを大幅に減少させることができます。

確かに、OSコマンドインジェクションの脆弱性はさまざまな形で発生する可能性があります。以下に、別の脆弱な実装例とその攻撃手法を紹介します。

---

### **PHP**

**脆弱なコード:**
```php
<?php
$dir = $_POST['directory'];
$output = shell_exec("cd " . $dir . " && ls -la");
echo "<pre>$output</pre>";
?>
```

**攻撃手法:**
このコードは、ユーザーが指定したディレクトリに移動し、そのディレクトリの内容を表示します。`directory`パラメータに`"; rm -rf / ;"`を入力することで、次のように実行されます。

```bash
cd /target_directory && ls -la; rm -rf /;
```

これにより、システム全体が削除される危険があります。

---

### **Python**

**脆弱なコード:**
```python
import subprocess

command = input("Enter your command: ")
subprocess.run(command, shell=True)
```

**攻撃手法:**
このコードは、ユーザーが入力したコマンドをそのままシェルに渡して実行します。`command`に`"; shutdown now"`などを入力することで、システムがシャットダウンされる可能性があります。

---

### **Node.js**

**脆弱なコード:**
```javascript
const { execSync } = require('child_process');

const userCommand = req.body.command;
const result = execSync(userCommand);
res.send(result.toString());
```

**攻撃手法:**
ユーザーが`command`として`"; cat /etc/passwd"`を送信すると、システムのパスワードファイルが漏洩する可能性があります。

---

### **Java**

**脆弱なコード:**
```java
import java.io.*;

public class VulnerableApp {
    public static void main(String[] args) {
        String command = args[0];
        try {
            Process proc = Runtime.getRuntime().exec(command);
            BufferedReader reader = new BufferedReader(new InputStreamReader(proc.getInputStream()));
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

**攻撃手法:**
`command`に`"; del C:\\Windows\\System32"`を指定することで、Windowsの重要なシステムファイルが削除される危険があります。

---

### **Perl**

**脆弱なコード:**
```perl
#!/usr/bin/perl

print "Enter the file to view: ";
my $file = <STDIN>;
chomp($file);
system("cat $file");
```

**攻撃手法:**
`file`に`"; rm -rf /"`を入力すると、システム全体が削除される可能性があります。

---

### **防止策の追加ポイント**

上記の例を防ぐために、以下の点も重要です。

1. **`shell=True`の使用を避ける**（Pythonの場合）  
   シェルを経由せずにコマンドを実行することで、コマンドインジェクションのリスクを回避します。

2. **コマンド列の直接入力を避ける**  
   例えば、Javaの例では`Runtime.getRuntime().exec()`の代わりに、`ProcessBuilder`を利用し、コマンドと引数を明確に分けて渡すようにします。

3. **ファイルパスのホワイトリスト化**  
   パスやコマンドの入力を受け取る際には、ホワイトリストを用いて許可された値のみを使用できるようにします。

4. **入力のサニタイズとエスケープ**  
   ユーザーからの入力を適切にサニタイズし、エスケープすることで、シェルの特殊文字を無効化します。

これらの対策を組み合わせることで、OSコマンドインジェクションのリスクをより効果的に防ぐことが可能です。
