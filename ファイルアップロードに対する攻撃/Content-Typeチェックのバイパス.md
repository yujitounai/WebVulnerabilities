---
title: Content-Typeチェックのバイパス
---

### ファイルアップロードにおけるコンテントタイプ（Content-Type）チェックのバイパス手法について

#### 概要
ファイルアップロード機能では、Content-Typeヘッダーに基づいてファイルの種類を判定することがあります。しかし、このContent-Typeヘッダーはクライアント（攻撃者）が自由に設定可能なため、信頼性が低いです。攻撃者は、このヘッダーを偽装して、意図しないファイル形式をサーバーにアップロードし、任意のスクリプト実行やデータ漏洩を引き起こす可能性があります。

#### 言語別の脆弱性を持つコードの例と攻撃手法

1. **PHP**
   - **脆弱性を持つコードの例:**
     ```php
     $allowedTypes = ['image/jpeg', 'image/png'];
     $fileType = $_FILES['file']['type'];

     if (in_array($fileType, $allowedTypes)) {
         move_uploaded_file($_FILES['file']['tmp_name'], 'uploads/' . $_FILES['file']['name']);
     }
     ```
   - **攻撃手法:**
     攻撃者は、PHPアプリケーションに任意のスクリプトをアップロードする際、`Content-Type: image/jpeg` と偽装してサーバーに送信します。サーバー側では、`$_FILES['file']['type']` に依存してチェックを行うため、実際のファイルがPHPスクリプトであっても、画像として扱われてしまいます。たとえば、ファイル名を `shell.php` としてアップロードしても、サーバーはその内容を確認せずに保存してしまいます。

2. **Python (Flask/Django)**
   - **脆弱性を持つコードの例:**
     ```python
     ALLOWED_TYPES = {'image/jpeg', 'image/png'}

     if 'file' in request.files:
         file = request.files['file']
         if file.content_type in ALLOWED_TYPES:
             file.save(os.path.join(app.config['UPLOAD_FOLDER'], file.filename))
     ```
   - **攻撃手法:**
     Pythonでも、攻撃者がリクエストヘッダーの `Content-Type` を自由に設定できます。たとえば、実際にはスクリプトコードが含まれたファイルに `Content-Type: image/png` を指定し、サーバーに画像として認識させることが可能です。

3. **Node.js (Express)**
   - **脆弱性を持つコードの例:**
     ```javascript
     const multer = require('multer');
     const upload = multer({
         fileFilter: function (req, file, cb) {
             const allowedTypes = /jpeg|png/;
             if (allowedTypes.test(file.mimetype)) {
                 return cb(null, true);
             }
             cb('Error: Invalid file type!');
         }
     });

     app.post('/upload', upload.single('file'), (req, res) => {
         res.send('File uploaded!');
     });
     ```
   - **攻撃手法:**
     Node.jsでも、攻撃者が `file.mimetype` に偽の値（例えば、`image/jpeg`）を設定して送信することでバイパスが可能です。サーバーはContent-Typeヘッダーに依存してチェックを行うため、意図しないファイル形式が受け入れられるリスクがあります。

4. **Java (Spring)**
   - **脆弱性を持つコードの例:**
     ```java
     @PostMapping("/upload")
     public String handleFileUpload(@RequestParam("file") MultipartFile file) {
         String fileType = file.getContentType();
         List<String> allowedTypes = Arrays.asList("image/jpeg", "image/png");
         if (allowedTypes.contains(fileType)) {
             file.transferTo(new File("uploads/" + file.getOriginalFilename()));
             return "File uploaded!";
         } else {
             return "Invalid file type!";
         }
     }
     ```
   - **攻撃手法:**
     Javaでも、クライアントが `Content-Type: image/jpeg` と偽装することで、スクリプトファイルが画像としてアップロードされるケースがあります。サーバーがContent-Typeヘッダーに依存していると、実際のファイル形式にかかわらず処理されてしまいます。

#### 言語による差異
- 全ての言語において、Content-Typeヘッダーはクライアント側で自由に設定可能であるため、信頼性は低いです。そのため、すべての言語でバイパス攻撃が成立する可能性があります。

#### 防止策
1. **Content-Typeヘッダーに依存しない検証**
   サーバー側でファイルのMIMEタイプや拡張子を検証する際、Content-Typeヘッダーに依存せず、ファイルの内容を検査します。たとえば、画像であれば実際に画像ライブラリを使って解析できるか確認します。

2. **ファイル内容のサーバー側検証**
   画像ファイルの場合、サーバー側でファイルの内容が正しい形式であるかどうかを検証します。例えば、画像の場合は画像ライブラリを使い、ファイルが正当な画像フォーマットであるかを確認します。

3. **実行可能な場所へのファイル保存を避ける**
   アップロードされたファイルをスクリプトが実行されるディレクトリに保存しないようにします。たとえば、アップロードされたファイルはWeb公開ディレクトリ外に保存し、必要に応じて読み取り専用で提供します。

4. **ファイル名の正規化と安全な保存**
   ファイル名に予期しない文字列やパターンが含まれないように正規化します。また、`secure_filename()` などの方法でファイル名を安全に処理します。

拡張子チェックと同様に、Content-Typeのチェックだけに頼らず、複数の方法でファイルの正当性を検証することが重要です。
