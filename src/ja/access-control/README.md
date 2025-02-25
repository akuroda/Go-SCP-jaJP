アクセスコントロール
==============

アクセスコントロールの最初のステップは、信頼できるシステム・オブジェクトだけを使ってアクセス認可の判断することです。

[セッション管理][3]の例では JWT（JSON Web Tokens：サーバーサイドでセッション・トークンを生成するしくみ）を使用しています。

```go
// create a JWT and put in the clients cookie
func setToken(res http.ResponseWriter, req *http.Request) {
    //30m Expiration for non-sensitive applications - OWASP
    expireToken := time.Now().Add(time.Minute * 30).Unix()
    expireCookie := time.Now().Add(time.Minute * 30)

    //token Claims
    claims := Claims{
        {...}
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    signedToken, _ := token.SignedString([]byte("secret"))
```


トークンを保存して使用することで、ユーザーのバリデーションやアクセスコントロールモデルの強制ができます。

アクセス認可に使用するコンポーネントは、サイト全体で単一のものであるべきです。
ここでのコンポーネントは、外部の認証サービスを呼び出すライブラリも含まれます。

失敗した場合、アクセス制御は安全に失敗する必要があります。Go では `defer` を使って実現できます。詳しくは、このドキュメントの [エラーログ][1] のセクションを参照してください。

アプリケーションが構成情報にアクセスできない時は、すべてのアクセスを拒否しましょう。

サーバーサイドのスクリプトや、AJAX や flash ようなクライアントサイドからのリクエストを含む、あらゆるリクエストに対して認可制御は実施されるべきです。

また、認可制御のような特権ロジックをアプリケーションコードのほかの部分から適切に分離することも重要です。

そのほかの重要な操作で、不正なユーザーによるアクセスを防止するためにアクセス制御が必要なものは以下の通りです。

* ファイルやそのほかのリソース
* 保護された URL
* 保護された関数
* オブジェクトの直接参照
* サービス
* アプリケーションデータ
* ユーザーやデータの属性、ポリシー情報

提示した例では、単純なオブジェクトの直接参照がテストされます。このコード
は、[セッション管理のサンプル][2]をもとに作成されています。

このようなアクセス制御を実装する場合、サーバーサイドとプレゼンテーション層のアクセスコントロールのルールが同じであることを確かめることが大事です。

状態に関するデータをクライアント側に保存する必要がある場合、改ざんを防ぐために、暗号化や完全性チェックを行うことが必要です。

アプリケーションロジックの流れが、ビジネスルールに適合している必要があります。

トランザクションを扱う場合、1 人のユーザーまたは 1 つのデバイスが一定の時間に実行できるトランザクション数の制限はビジネス要件以上である必要がありつつ、DoS 攻撃を防げる程度には低くある必要があります。

`referer` HTTP ヘッダのみを認可のバリデーションに使うのは不十分であり、あくまで補助的なものとして使用されるべきことは注意です。

長時間の認証セッションに関しては、アプリケーションは定期的にユーザーの権限を再評価して、ユーザーの権限に変更がないことを確認してください。
権限が変更されている場合は、ユーザーを強制的にログアウトさせ、再認証させてください。

またユーザーアカウントを監査して安全な手続きで管理しましょう。(例：ユーザーのアカウントをパスワード失効から 30 日後に無効化する）

またユーザーの認可が取り消された時のために、アプリケーションはアカウントの無効化とセッションの停止ができる必要があります。(例：役職の変更や、雇用形態の変更など）。

外部サービスアカウントや外部サービスと接続するためのアカウントをサポートする場合、これらのアカウントには最低限のレベルの権限を与えましょう。

[1]: ../error-handling-logging/error-handling.md
[2]: URL.go
[3]: ../session-management/README.md
