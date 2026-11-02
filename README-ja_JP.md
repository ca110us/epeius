# Epeius

[English](./README.md) | [简体中文](./README-zh_CN.md) | 日本語

**Cloudflare Workers 上で動作する WebSocket–TCP 転送ツール。**

Epeius は、WebSocket 経由で Trojan プロトコルのリクエストを受け取り、Workers Sockets API を使って TCP の接続先へ通信を転送する軽量ネットワークプロキシです。プロトコルの解析、接続処理、双方向のデータ転送を、単一の JavaScript Worker で行います。

## 仕組み

```text
Trojan 対応クライアント
        |
   WebSocket over TLS
        |
Cloudflare Worker (Epeius)
        |
   アウトバウンド TCP 接続
        |
    接続先サービス
```

1. クライアントが Worker への WebSocket 接続を開き、Trojan リクエストを送信します。
2. Worker が認証フィールドを検証し、接続先のアドレスとポートを読み取ります。
3. `cloudflare:sockets` で TCP 接続を開き、両端の間でデータを双方向に転送します。

TLS はクライアントと Worker の間の接続を保護します。Worker から接続先へは TCP ソケットを開きます。転送されるデータ自体が暗号化されるかどうかは、アプリケーションプロトコルによります。

## クイックスタート

Cloudflare アカウントと、**Trojan + WebSocket + TLS** に対応したクライアントが必要です。

### 1. パスワードを用意する

強度の高いパスワードを選び、その SHA-224 ダイジェストをローカルで計算します。Python 3 を使う場合は、次のコマンドを実行できます。

```sh
python3 -c 'import getpass, hashlib; print(hashlib.sha224(getpass.getpass("Password: ").encode("utf-8")).hexdigest())'
```

クライアントには元のパスワードを使用します。Worker には元のパスワードではなく、計算結果の **56 文字の小文字 16 進数ダイジェスト**を設定してください。パスワードとダイジェストは、どちらも認証情報として適切に管理してください。

### 2. Worker をデプロイする

1. Cloudflare ダッシュボードで Worker を作成します。
2. `SHA224PASS` という名前の Worker [シークレット](https://developers.cloudflare.com/workers/configuration/secrets/)を追加し、手順 1 で生成したダイジェストを設定して、この設定をデプロイします。
3. コードエディターを開き、初期コードを [src/worker.js](./src/worker.js) の内容に置き換えてデプロイします。
4. Worker のホスト名を使用します。必要に応じて[カスタムドメイン](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/)を設定することもできます。

プロキシを使用する前に、自分の `SHA224PASS` を設定してください。未設定の場合は、ソースコードに含まれる公開済みのサンプル認証情報が使われます。

### 3. クライアントを設定する

| 設定項目 | 値 |
| --- | --- |
| プロトコル | Trojan |
| サーバー | Worker のホスト名。`https://` は含めません |
| ポート | `443` |
| パスワード | 元のパスワード。SHA-224 ダイジェストでは**ありません** |
| トランスポート | WebSocket |
| WebSocket パス | `/` |
| TLS | 有効。SNI と WebSocket の Host ヘッダーには同じ Worker のホスト名を使用します |

`https://<your-worker-hostname>/link` にアクセスすると、クライアント URI のテンプレートも取得できます。テンプレートには常に固定のプレースホルダー `ca110us` が含まれるため、インポート前に自分のパスワードへ置き換えてください。このエンドポイントは設定済みのパスワードを読み取ったり表示したりせず、`SHA224PASS` を変更してもテンプレートは変わりません。

パスワードに URI の予約文字が含まれる場合は、パスワード部分をパーセントエンコードしてからリンクに挿入するか、上の表に従ってクライアントを手動で設定してください。

## 設定

| 名前 | 用途 |
| --- | --- |
| `SHA224PASS` | クライアントのパスワードの SHA-224 ダイジェスト。Worker シークレットとして設定すると、ソース内の `sha224Password` より優先されます。 |
| `PROXYIP` | 任意設定の代替 TCP 接続先ホスト名または IP アドレス。適切な代替接続先がない場合は、未設定のままにしてください。 |

最初の接続先からの読み取りが終了または失敗し、それまでにデータを一度も受信していない場合、代替接続を試みます。`PROXYIP` が設定されていれば元の接続先ポートを使ってそこへ接続し、未設定であれば元のアドレスに再接続します。これは直接の TCP 接続であり、HTTP や SOCKS プロキシのハンドシェイクは行いません。

## 対応範囲と制限

- TCP `CONNECT` リクエストのみ対応しています。UDP は未実装です。
- リクエストの解析は IPv4、IPv6、ドメイン名に対応しています。実際の接続可否は [Cloudflare の TCP ソケット制限](https://developers.cloudflare.com/workers/runtime-apis/tcp-sockets/#considerations)に従います。
- クライアントには WebSocket トランスポートへの対応が必要です。通常の Trojan over TCP/TLS のみに対応したクライアントは、この Worker に直接接続できません。
- 認証には単一のパスワードダイジェストを使用します。ユーザー管理、ユーザーごとの利用上限、アプリケーション側のレート制限は含まれていません。

アクセス権限のあるネットワークリソースに対して、ホスティングサービスの利用規約に従って使用してください。

## 謝辞

本実装は [zizifn/edgetunnel](https://github.com/zizifn/edgetunnel) を参考にしています。

## ライセンス

GNU GPL v2 のライセンス本文は [LICENSE](./LICENSE) を参照してください。
