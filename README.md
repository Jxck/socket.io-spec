Socket.IO protocol
==================

<!--
The present document describes the protocol and encoding a client/server pair
must follow to establish a successful Socket.IO connection.
-->
このドキュメントは、クライアントとサーバが Socket.IO コネクションを接続するために、
満たすべきプロトコルとエンコーディングについての仕様です。

<!--
## Protocol versions
-->
## プロトコルのバージョン

<!--
The present document describes the latest version of the protocol, **1**.
-->
このドキュメントは、最新であるプロトコルバージョン **1** について記述しています。

<!--
Versions are a single integer incremented with each revision of the protocol.
-->
バージョン番号は単一の整数であり、プロトコルのバージョンアップ毎にインクリメントされていきます。

<!--
## Overview
-->
## 概要

<!--
Socket.IO aims to bring a WebSocket-like API to many browsers and devices, with
some specific features to help with the creation of real-world realtime
applications and games.
-->
Socket.IO は、実際のリアルタイムアプリケーションやゲーム開発の手助けするための
いくつかの機能をもつ、WebSocket-like な API を、
多くのブラウザやデバイス上で実現することを目的としています。

<!--
- Multiple transport support (old user agents, mobile browsers, etc).
- Multiple sockets under the same connection (namespaces).
- Disconnection detection through heartbeats.
- Optional acknoledgments.
- Reconnection support with buffering (ideal for mobile devices or bad networks)
- Lightweight protocol that sits on top of HTTP.
-->
- (古いブラウザやモバイルに対応するための)複数の通信方法サポート
- 同一接続上での複数ソケットの実現(名前空間)
- ハートビートによるコネクションの自動切断
- 送達確認
- (電波の悪い環境やモバイルのための)バッファ付き再接続のサポート
- HTTP をベースとした軽量プロトコル


<!--
### Anatomy of a Socket.IO socket
-->
### Socket.IO のソケットの構造

<!--
A Socket.IO client first decides on a transport to utilize to connect. 
-->
Socket.io のクライアントは最初に接続に利用する通信方法を決定します。

<!--
The state of the Socket.IO socket can be `disconnected`, `disconnecting`,
`connected` and `connecting`.
-->
Socket.IO のソケットは `disconnected`, `disconnecting`, `connected`, `connecting`
の状態を持ちます。

<!--
The transport connection can be `closed`, `closing`, `open`, and `opening`.
-->
通信コネクションには `closed`, `closing`, `open`, and `opening` があります。

<!--
A simple HTTP handshake takes place at the beginning of a Socket.IO connection.
The handshake, if successful, results in the client receiving:
-->
Socket.IO の接続の最初には、シンプルな HTTP ハンドシェイクが実行されます。
ハンドシェイクが成功すると、以下の結果がクライアントに送られます。

<!--
- A session id that will be given for the transport to open connections.
- A number of seconds within which a heartbeat is expected (`heartbeat
timeout`)
- A number of seconds after the transport connection is closed when the socket
is considered disconnected if the transport connection is not reopened (`close
timeout`).
-->
- 通信が接続を開始する時に渡される session id
- ハードビートの送信が期待される間隔の秒数(`heartbeat timeout`)
- 通信コネクションが応答しなくなり、ソケットが閉じられたと判断して
通信コネクションを閉じるまでの秒数(`close timeout`)

<!--
At this point the socket is considered connected, and the transport is signaled
to open the connection.
-->
この時点で、ソケットは接続されていると考えられ、通信コネクションにコネクションを
オープンするようにシグナルがわたります。

<!--
If the transport connection is closed, both ends are to buffer messages and
then frame them appropriately for them to be sent as a batch when the
connection resumes.
-->
通信コネクションが閉じられたら、両端(サーバ/クライアント)はメッセージをバッファし、
コネクションが再開したときに、まとめて送れるように調度良いサイズのフレームにまとめます。

<!--
If the connection is not resumed within the negotiated timeout the socket is
considered disconnected. At this point the client might decide to reconnect the
socket, which implies a new handshake.
-->
タイムアウト期間内にコネクションが再開しなかったら、ソケットは通信が切断したと判断します。
この時点で、クライアントは新しいハンドシェイクによってソケットの再接続(reconnect)を開始します。


<!--
## Socket.IO HTTP requests
-->
## Socket.IO HTTP リクエスト

<!--
Socket.IO HTTP URIs take the form of:
-->
Socket.IO の HTTP URI は以下のフォーマットです。

    [scheme] '://' [host] '/' [namespace] '/' [protocol version] '/' [transport id] '/' [session id] '/' ( '?' [query] )

<!--
Only the methods `GET` and `POST` are utilized (for the sake of compatibility
with old user agents), and their usage varies according to each transport.
-->
(古い User Agent との互換のため)使用されるのは `GET` と `POST` メソッドだけです。
実際の使用方法は通信方法によって違います。

<!--
The main transport connection is always a `GET` request.
-->
通信コネクションが使うメインのメソッドは常に `GET` です。

<!--
### URI scheme
-->
### URI スキーマ

<!--
The URI scheme is decided based on whether the client requires a secure connection
or not. Defaults to `http`, but `https` is the recommended one.
-->
URI スキーマは、クライアントがセキュアな通信を要求しているかに応じて決定されます。
デフォルトは `http` ですが、推奨は `https` です。

<!--
### URI host
-->
### URI ホスト

<!--
The host where the Socket.IO server is located. In the browser environment, it
defaults to the host that runs the page where the client is loaded (`location.host`)
-->
Socket.IO サーバが存在するホストです。ブラウザ環境では、この値はデフォルトで
クライアントのページを配信したサーバ(`location.host`)と同じになります。

<!--
### Namespace
-->
### 名前空間(Namespace)

<!--
The connecting client has to provide the namespace where the Socket.IO requests
are intercepted.
-->
接続しているクライアントは、 Socket.IO リクエストを区分する名前空間を提供します。

<!--
This defaults to `socket.io` for all client and server distributions.
-->
この値は全てのクライアントとサーバに対して、デフォルトで `socket.io` という値が配布されます。


<!--
### Protocol version
-->
### プロトコルバージョン

<!--
Each client should ship with the revision ID it supports, available as a public
interface to developers.
-->
クライアントは個々に、自身がサポートするリビジョン ID を、開発者がアクセスできる
パブリックなインタフェースとして持つ必要があります。

<!--
For example, the browser client supports `io.protocolVersion`.
-->
例えば、ブラウザがサポートするプロトコルのバージョンは `io.protocolVersion` で取得できます。

<!--
### Transport ID
-->
### 通信 ID(Transport ID)

<!--
The following transports are supported:
-->
以下の通信方法がサポートされています。

- `xhr-polling`
- `xhr-multipart`
- `htmlfile`
- `websocket`
- `flashsocket`
- `jsonp-polling`

<!--
The client first figures out what transport to use. Usually this occurs in the
browser, utilizing [feature detection](http://mzl.la/6tm7Aj).
-->
クライアントは最初にどの通信を使用するかを決定します。
通常、ブラウザ内で [feature detection](http://mzl.la/6tm7Aj) を用いて決定されます。

<!--
User-defined transports are allowed.
-->
ユーザが定義した通信方法を使用することもできます。

<!--
### Query
-->
### クエリ

<!--
The query component (eg: `?token=48737481747&`) is not present on all URLs. 
Certain query keys are reserved by Socket.IO:
-->
クエリコンポーネント (eg: `?token=48737481747&`) は全ての URL にあるわけではありません。
特定の key は Socket.IO によって予約されています。

<!--
* `t`: Contains a timestamp, only used to bypass caching on certain old UAs.
* `disconnect`: Triggers a disconnection.
-->
* `t`: タイムスタンプを保持します。これは特定の古い UA のキャッシュをバイパスするためだけに使用します。
* `disconnect`: 切断を要求するトリガーです。

<!--
User-defined query components are allowed. For example,
`?t=1238141910&token=mytoken` is a valid query).
-->
ユーザ定義のクエリコンポーネントも以下のように使用可能です。
例えば `?t=1238141910&token=mytoken` は有効なクエリです。


<!--
## Handshake
-->
## ハンドシェイク

<!--
The client will perform an initial HTTP POST request like the following
-->
クライアントは以下のような HTTP POST リクエストを発行します。

    http://example.com/socket.io/1/

<!--
The absence of the `tranport id` and `session id` segments will signal the server
this is a new, non-handshaken connection.
-->
`transport id` と `session id` が無かった場合、サーバはこの要求をハンドシェイクが
済んでいない新しいコネクションと判断します。

<!--
The server can respond in three different ways:
-->
サーバは異なる 3 種のレスポンスを返すことができます。

  - 401 Unauthorized

<!--
  If the server refuses to authorize the client to connect, based on the supplied
  information (eg: `Cookie` header or custom query components).
-->
サーバが与えられた情報( `Cookie` ヘッダやカスタムクエリコンポーネントなど)を用いて
クライアントの認証を拒否した場合。

<!--
  No response body is required.
-->
レスポンスボディーは必要ありません。

  - 503 Service Unavailable

<!--
  If the server refuses the connection for any reason (eg: overload).
-->
もしサーバが何らかの理由(過負荷など)で接続を拒否した場合。

<!--
  No response body is required.
-->
レスポンスボディーは必要ありません。

  - 200 OK

<!--
  The handshake was successful.
-->
ハンドシェイクが成功した場合。

<!--
  The body of the response should contain the session id (`sid`) given to the
  client, followed by the heartbeat timeout, the connection closing timeout,
  and the list of supported transports separated by `:`
-->
レスポンスボディは、クライアントに付与するセッション ID (`sid`)、
それに続く、 heartbeat timeout、 connection closing timeout、
そして、サポートする通信方法の `:` 区切りのリストを含む必要があります。


<!--
  The absence of a heartbeat timeout ('') is interpreted as the server and
  client not expecting heartbeats.
-->
ハートビートのタイムアウトが含まれていなかった場合('')は、
サーバもクライアントもハートビートの実行を期待しないものと解釈されます。

<!--
  For example `4d4f185e96a7b:15:10:websocket,xhr-polling`.
-->
  例 `4d4f185e96a7b:15:10:websocket,xhr-polling`.


<!--
## Transport connection
-->
## 通信コネクション

<!--
Once the handshake request-response cycle is complete (and it ended with success),
a new connection is opened by the transport that was negotiated, with a `GET`
HTTP request.
-->
一旦ハンドシェイクが完了しそれが成功だった場合、
ハンドシェイク中に決定した通信手段による HTTP の `GET` リクエストで
コネクションがオープンされ実行されます。

<!--
The transport *can* modify the URI if the transport requires it, as long as no
information is lost. For example, if `websocket` is accepted as the transport,
and the connection was secure, the URI for the transport connection will become:
-->
通信コネクションは必要に応じて URI を変更することが *可能* です。
ただし、情報が失われない場合に限ります。
例えば、ハンドシェイクの結果通信手方法としてセキュアな `websocket` が採用された場合は、
URI は以下のようになります。

    wss://example.com/socket.io/1/websocket/4d4f185e96a7b

<!--
The URI still contains all the information required by Socket.IO to continue the
message exchange (protocol security, namespace, protocol version, transport, etc).
-->
URI は Socket.IO がメッセージをやり取りする上で必要な全ての情報を含んでいます。
(protocol security, namespace, protocol version, transport, など).

<!--
Messages can be sent and received by following this convention. **How** the messages
are encoded and framed depends on each transport, but generally boils down to
whether the transport has built-in framing (unidiretionally and/or bidirectionally).
-->
メッセージは、次に述べる取り決めにより送受信されます。 **どのように** メッセージが
エンコードされ、フレームにまとめられるかは、通信方法に依存します。
しかし、通常通信がビルトインでもつフレーム技術(単/双方向) が使われるでしょう。

<!--
### Unidirectional transports
-->
### 一方向のみの通信

<!--
Transports that initialize unidirectional connections (where the server can
write to the client but not vice-versa), should perform `POST` requests to send
data back to the server to the same endpoint URI.
-->
通信コネクションが、単方向通信として初期化される(サーバはクライアントに送信できるが、逆はできない)
場合、 `POST` リクエストを同じエンドポイント URI のサーバに送り返す挙動をとる必要があります。


<!--
## Messages
-->
## メッセージ

<!--
### Framing
-->
### フレーム処理

<!--
Certain transports, like `websocket` or `flashsocket`, have built-in lightweight
framing mechanisms for sending and receiving messages.
-->
`websocket` や `flashsocket` といった特定の通信方法は、
メッセージを送受信するための軽量なフレーミングのメカニズムを標準で持っています。

<!--
For `xhr-multipart`, the built-in MIME framing is used for the sake of consistency.
-->
例えば `xhr-multipart` は、一貫性のために標準の MIME フレーミングを使用します。

<!--
When no built-in lightweight framing is available, and multiple messages need to be
delivered (i.e: buffered messages), the following is used:
-->
ビルトインの軽量フレーミングがない場合、そしてメッセージが多重化される場合(i.e: メッセージのバッファリング)
以下のフォーマットが使われます。

    `\ufffd` [message lenth] `\ufffd`

<!--
Transports where the framing overhead is expensive (ie: when the xhr-polling
transport tries to send data to the server).
-->
これは、通信におけるフレーミングのオーバーヘッドが大きいところで使用されます。
(ie: xhr-polling がサーバにデータを送信する場合)

### Encoding

<!--
Messages have to be encoded before they're sent. The structure of a message is
as follows:
-->
メッセージは送信される前にエンコードされる必要があります。メッセージは以下のような構造です。

    [message type] ':' [message id ('+')] ':' [message endpoint] (':' [message data]) 

<!--
The message type is a single digit integer.
-->
message type は一桁の整数です。

<!--
The message id is an incremental integer, required for ACKs (can be ommitted).
-->
message id はインクリメンタルな整数です。ACK(省略可能)のために必要です。

<!--
If the message id is followed by a `+`, the ACK is not handled by socket.io,
-->
もし message id の後に `+` が続く場合、ACK は Socket.IO に処理されません。

<!--
but by the user instead.
-->
しかし、ユーザが処理する場合はその限りではありません。

<!--
Socket.IO has built-in support for multiple channels of communication (which we
call "multiple sockets"). Each socket is identified by an endpoint (can be
omitted).
-->
Socket.IO  は標準で multiple channels をサポートします。("multiple sockets")
各々のソケットはエンドポイントで識別されます(省略可能)


### (`0`) Disconnect

<!--
Signals disconnection. If no endpoint is specified, disconnects the entire
socket.
-->
切断を知らせるシグナルです。もし特定のエンドポイントが指定されなかった場合、全てのソケットを切断します。

Examples:

<!--
- Disconnect a socket connected to the `/test` endpoint.
-->
- `/test` というエンドポイントに接続するソケットを切断します。

      0::/test

<!--
- Disconnect the whole socket
-->
- 全てのソケットを切断します。

      0

### (`1`) Connect

<!--
Only used for multiple sockets. Signals a connection to the endpoint.
Once the server receives it, it's echoed back to the client.
-->
multiple sockets を利用している場合に限り使用します。
エンドポイントにシグナルが送信されます。
一旦サーバがそれを受信したら、クライアントにエコーバックします。

<!--
Example, if the client is trying to connect to the endpoint /test, a message
like this will be delivered:
-->
例えば、クライアントが /test エンドポイントに接続を試みる場合、以下のようなメッセージが送信されます。

    '1::' [path] [query]

Example:

    1::/test?my=param

<!--
To acknowledge the connection, the server echoes back the message. Otherwise,
the server might want to respond with a error packet.
-->
接続を承認するため、サーバはメッセージをエコーバックします。
そうでなければ、サーバはエラーパケットを送信する可能性もあります。

### (`2`) Heartbeat

<!--
Sends a heartbeat. Heartbeats must be sent within the interval negotiated with
the server. It's up to the client to decide the padding (for example, if the
heartbeat timeout negogiated with the server is 20s, the client might want to
send a heartbeat evert 15s).
-->
ハートビートを送信します。ハートビードはサーバと取り決めた感覚で送信される必要があります。
クライアントは余分な値を決定します。
(もしサーバとの間で heartbeat timeout が 20 秒と決まった場合、
クライアントは、 15 秒間隔での送信を繰り返すでしょう)

### (`3`) Message

    '3:' [message id ('+')] ':' [message endpoint] ':' [data]

<!--
A regular message.
-->
標準的なメッセージです。

    3:1::blabla

### (`4`) JSON Message

    '4:' [message id ('+')] ':' [message endpoint] ':' [json]

<!--
A JSON encoded message.
-->
JSON エンコードされたメッセージです。

    4:1::{"a":"b"}

### (`5`) Event

    '5:' [message id ('+')] ':' [message endpoint] ':' [json encoded event]

<!--
An event is like a json message, but has mandatory `name` and `args` fields.
`name` is a string and `args` an array.
-->
イベントは JSON メッセージと似ています。しかし、 `name` と `args` フィールドを
含む必要があります。 `name` は文字列、 `args` は配列です。

<!--
The event names
-->
以下のイベント名は予約されています。

    'message'
    'connect'
    'disconnect'
    'open'
    'close'
    'error'
    'retry'
    'reconnect'

<!--
are reserved, and cannot be used by clients or servers with this message type.
-->
クライアント/サーバともにこれらをメッセージタイプとすることはできません。

<!--
### (`6`) ACK
-->
### (`6`) 送達確認(ACK)

    '6:::' [message id] '+' [data]

<!--
An acknowledgment contains the message id as the message data. If a `+` sign
follows the message id, it's treated as an event message packet.
-->
送達確認は message data としてmessage id を含みます。
もし message id の後に `+` が続いた場合は、イベントメッセージのパケットとして扱われます。

<!--
Example 1: simple acknowledgement
-->
Example 1: 単純な送達確認

    6:::4

<!--
Example 2: complex acknowledgement
-->
Example 2: イベントメッセージを含む送達確認

    6:::4+["A","B"]

### (`7`) Error

    '7::' [endpoint] ':' [reason] '+' [advice]

<!--
For example, if a connection to a sub-socket is unauthorized.
-->
例えば、サブソケットへのコネクションが認証されなかった場合。

### (`8`) Noop

<!--
No operation. Used for example to close a poll after the polling duration times
out.
-->
No operation(なにもしない)。例えば、ポーリングがタイムアウトした時、接続を閉じる場合などに使います。

<!--
## Forced socket disconnection
-->
## ソケットの強制切断

<!--
A Socket.IO server must provide an endpoint to force the disconnection of the
socket.
-->
Socket.IO サーバは、ソケットを強制切断するためのエンドポイントを提供する必要があります。

<!--
While closing the transport connection is enough to trigger a disconnection, it
sometimes is desireable to make sure no timeouts are activated and the
disconnection events fire immediately.
-->
通信コネクションを閉じるには、 disconnection イベントをトリガーすれば十分です。
タイムアウトが発生しないのを確認し、 disconnection イベントがすぐに発生することが
望ましい場合もあります。

    http://example.com/socket.io/1/xhr-polling/812738127387123?disconnect

<!--
The server must respond with `200 OK`, or `500` if a problem is detected.
-->
サーバは `200 OK` か、問題が発生しているのであれば `500` を返す必要があります。
