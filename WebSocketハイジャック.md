---
title: WebSocketハイジャック
---

**WebSocketハイジャック攻撃について**

### **概要**

WebSocketハイジャック攻撃（WebSocket Hijacking）は、攻撃者がWebSocket通信を乗っ取って、不正な操作を行ったり、データを盗み出したりする攻撃手法です。WebSocketは、クライアントとサーバー間で双方向のリアルタイム通信を行うためのプロトコルですが、不適切な認証やセッション管理が行われている場合、攻撃者がセッションをハイジャックすることが可能です。この攻撃は、クロスサイトスクリプティング（XSS）やクロスサイトリクエストフォージェリ（CSRF）と組み合わせて行われることが多いです。

---

### **脆弱性を持つコードの例と攻撃手法**

#### **PHP（Ratchet使用）**

**脆弱なコード:**
```php
<?php
// WebSocketサーバー側のコード
use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

class Chat implements MessageComponentInterface {
    public function onOpen(ConnectionInterface $conn) {
        // 接続を保存する
    }

    public function onMessage(ConnectionInterface $from, $msg) {
        // 受信したメッセージを他の接続にブロードキャストする
    }

    public function onClose(ConnectionInterface $conn) {
        // 接続が閉じられた時の処理
    }

    public function onError(ConnectionInterface $conn, \Exception $e) {
        // エラーハンドリング
        $conn->close();
    }
}
```

### **PHPのサンプルコード（ライブラリ不使用）**

```php
<?php
// WebSocketサーバーを簡単に実装する例（WebSocket対応サーバーが必要）
$address = '127.0.0.1';
$port = 8080;

$sock = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
socket_bind($sock, $address, $port);
socket_listen($sock);

while (true) {
    $client = socket_accept($sock);
    $request = socket_read($client, 5000);
    
    // ハンドシェイクを処理
    if (strpos($request, 'GET') !== false) {
        $headers = "HTTP/1.1 101 Switching Protocols\r\n";
        $headers .= "Upgrade: websocket\r\n";
        $headers .= "Connection: Upgrade\r\n";
        $headers .= "Sec-WebSocket-Accept: " . base64_encode(sha1(trim(explode('Sec-WebSocket-Key: ', $request)[1]), true)) . "\r\n\r\n";
        socket_write($client, $headers);
    }
    
    // メッセージの読み込みと返信
    while (true) {
        $data = socket_read($client, 5000);
        if ($data) {
            $response = "Received: " . $data;
            socket_write($client, $response);
        }
    }
    
    socket_close($client);
}
socket_close($sock);
```

**注意:** 上記のコードは、教育目的の非常に簡略化された例であり、実際のWebSocketサーバーとして運用するには不十分です。特に、セキュリティ対策やエラーハンドリングが欠けているため、本番環境での使用は推奨されません。

**攻撃手法:**
このコードは認証やセッション管理を行っていないため、攻撃者が同じWebSocketサーバーに接続して、他のユーザーのセッションを乗っ取ることが可能です。攻撃者は、正規ユーザーになりすまして、サーバーに不正なメッセージを送信できます。

#### **Python（Flask + Flask-SocketIO）**

**脆弱なコード:**
```python
from flask import Flask
from flask_socketio import SocketIO, send

app = Flask(__name__)
socketio = SocketIO(app)

@socketio.on('message')
def handleMessage(msg):
    send(msg, broadcast=True)

if __name__ == '__main__':
    socketio.run(app)
```

**攻撃手法:**
このコードには認証機構が実装されておらず、攻撃者はWebSocket接続をハイジャックして、任意のメッセージを送信したり、受信したりすることが可能です。

#### **Node.js**

**脆弱なコード:**
```javascript
const express = require('express');
const http = require('http');
const socketIo = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

io.on('connection', (socket) => {
    socket.on('message', (msg) => {
        io.emit('message', msg);
    });
});

server.listen(3000);
```

**攻撃手法:**
このコードにはユーザーの認証やセッション管理が欠如しており、攻撃者が任意のWebSocketクライアントを用いて、他のユーザーのセッションをハイジャックし、不正なメッセージを送信することが可能です。

#### **Java（Spring + Spring WebSocket）**

**脆弱なコード:**
```java
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

public class ChatWebSocketHandler extends TextWebSocketHandler {

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        session.sendMessage(new TextMessage("Hello, " + message.getPayload() + "!"));
    }
}
```

**攻撃手法:**
このコードはWebSocket接続に対して認証やセッション管理を行っておらず、攻撃者がWebSocket接続をハイジャックして、他のユーザーのセッションを不正に利用することが可能です。

---

### **防止策**

1. **WebSocket接続の認証とセッション管理:**
   - WebSocket接続を確立する前に、適切な認証を行い、セッションIDやトークンを使用して、各接続をユーザーと関連付けます。
   - 認証情報は、セキュアな方法でWebSocket接続に渡す必要があります（例えば、クッキーやWebSocketハンドシェイク時のカスタムヘッダー）。

2. **CSRFトークンの導入:**
   - クロスサイトリクエストフォージェリ（CSRF）攻撃を防ぐため、CSRFトークンを使用して、WebSocketリクエストが正当なクライアントからのものであることを検証します。

3. **接続の検証とタイムアウト:**
   - WebSocket接続の検証を行い、不正な接続やアイドル状態が続く接続をタイムアウトさせて切断します。

4. **入力データの検証とサニタイズ:**
   - WebSocketで受信するすべてのデータを検証し、期待された形式や内容に合致しているか確認します。また、サニタイズを行い、悪意のあるコードが実行されないようにします。

5. **SSL/TLSの使用:**
   - WebSocket通信を保護するために、SSL/TLSを使用して通信内容を暗号化し、通信の盗聴や改ざんを防ぎます。

6. **接続数やリソースの制限:**
   - WebSocketサーバーに対する接続数やリソースの使用を制限し、サービス拒否（DoS）攻撃に対する耐性を強化します。

---

**まとめ:**
WebSocketハイジャック攻撃は、WebSocket通信を乗っ取ることで、不正な操作や情報漏洩を引き起こす攻撃です。認証やセッション管理の実装、CSRFトークンの使用、データのサニタイズ、SSL/TLSの導入などの防止策を適用することで、この脆弱性を効果的に防ぐことが可能です。各プラットフォームや言語に共通する問題であるため、これらの対策を実施することが推奨されます。



### **攻撃手法:**
このような単純なWebSocketサーバーでは、セッション管理や認証が実装されていないため、攻撃者が任意のWebSocketクライアントを使って接続を確立し、不正なメッセージを送信することで、セッションをハイジャックする可能性があります。

### **防止策**
上記の防止策セクションで挙げた内容（認証とセッション管理の実装、CSRFトークンの導入、SSL/TLSの使用など）を適用することで、セキュリティを強化し、このような攻撃を防ぐことができます。