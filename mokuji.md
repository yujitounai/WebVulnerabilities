## 目次

[OSコマンドインジェクション](OSコマンドインジェクション)
OSコマンドインジェクション.md
コードインジェクション.md
サーバーサイドテンプレートインジェクション（SSTI）.md
リモートファイルインクルード.md
ローカルファイルインクルード.md
SSIインジェクション.md
デシリアライゼーション.md


SQLインジェクション.md
NoSQLインジェクション.md
XPathインジェクション.md
XML外部実体攻撃（XXE）.md
ディレクトリトラバーサル.md
サーバーサイドリクエストフォージェリ（SSRF）.md

ファイルアップロードによる攻撃.md
- Malicious File Upload 悪意のあるファイルアップロード
- Bypass Extension Check 拡張子チェックをバイパスする
- Bypass Content-Type Check コンテントタイプチェックをバイパスする
- EXIF Metadata not Removed from Images 画像からメタデータが除外されていない
- Missing File Size Check ファイルサイズチェックの不足
- Overwrite Web Server File Webサーバーファイルの上書き
- Path Traversal パストラバーサル

クロスサイトスクリプティング（XSS）.md
HTTPヘッダインジェクション.md
CRLF Injection CRLFインジェクション
ホストヘッダーインジェクション
オープンリダイレクト.md

セッションハイジャック
セッション固定.md
JWT（JSON Web Token）に対する攻撃.md

レースコンディション.md
TOCTOU.md

DoS
ReDoS.md

Request Smuggling.md
キャッシュデセプション.md
キャッシュポイズニング.md

WebSocketハイジャック.md
JSONPハイジャック.md
JSONハイジャック.md
プロトタイプ汚染.md

GraphQLの悪用

情報漏洩
Through Error Pages エラーページを通じて
Through Response Headers レスポンスヘッダを通じて
Through comments コメントを通じて
Through StackTrace/Debug messages スタックトレース/デバッグメッセージを通じて
Through direct request 直接リクエストを通じて
Through other HTTP Methods 他のHTTPメソッドを通じて
Through files ファイルを通じて
Misconfigurations 設定ミス
Unencrypted Communication HTTP 非暗号化通信HTTP
SSL/TLS Misconfigurations SSL/TLSの設定ミス
Missing/Misconfigured Security Headers 不足/誤設定されたセキュリティヘッダ
Missing Security Flags on Cookies クッキーのセキュリティフラグが不足
Missing Rate-Limiting レート制限の不足
OPTIONS/TRACE Methods Allowed OPTIONS/TRACEメソッドが許可されている
No custom pages defined for error pages エラーページにカスタムページが定義されていない
Directory Listing ディレクトリリスティング
Clickjacking クリックジャッキング

ユーザー名/メールアドレスの列挙
Through Login Error Message Discrepancy ログインエラーメッセージの不一致を通じて
Through Forget/Reset Password Functionality パスワード忘れ/リセット機能を通じて
Through Registration Form 登録フォームを通じて
Through Response Time Discrepancy 応答時間の不一致を通じて
Through Responses Size Discrepancy 応答サイズの不一致を通じて
Through Account Lockout Message アカウントロックアウトメッセージを通じて

---


Arbitrary File Read/Write Download 任意のファイルの読み取り/書き込み/ダウンロード



---

CSV/Formula Injection CSV/数式インジェクション