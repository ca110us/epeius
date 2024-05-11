# Epeius
[English](./README.md) | [简体中文](./README-zh_CN.md) | 日本語

サーバーレスアプローチを用いた Trojan のデプロイ

## クイックスタート
- Cloudflare Workers ダッシュボードで新しい Worker を作成します。
- [worker.js](./src/worker.js) のコードを Worker のコードエディタに貼り付けます。
- `sha224Password` は自分のパスワードに置き換えてください。[こちら](https://www.atatus.com/tools/sha224-to-hash)で生成できます。あるいは、後で Cloudflare Workers の設定で `SHA224PASS` 環境変数を追加することもできます。
- カスタムドメインを Worker にバインドする。
- `https://[YOUR_DOMAIN]/link` にアクセスし、`ca110us` をプレーンなパスワードに置き換える。

## サポート外
- UDP 🙅 (Cloudflare workers ランタイムはまだ UDP をサポートしていません)

## 免責事項
このプロジェクトは研究/調査目的のみのものです。利用者は法令遵守と倫理的行動に責任を負うものとする。著者は誤用に関するすべての責任を否認します。

## 参考
[zizifn/edgetunnel](https://github.com/zizifn/edgetunnel)
