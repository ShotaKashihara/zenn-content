---
title: "iOS 14 で Universal Links が動かないって？"
emoji: "🌏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS", "Swift", "UniversalLinks", "WWDC2020"]
published: false
---

週末は iOSDC Japan 2020 楽しかったですね。運営の方々おつかれさまでした👏

## iOS 14 の変更点

あなたの `apple-app-site-association` ファイルが Apple の CDN にキャッシュされるようになりました🎉

iOS 14 から、アプリインストール時の Universal Links の関連付けには `/.well-known/apple-app-site-association` ではなく `https://app-site-association.cdn-apple.com` という Apple が管理する CDN を参照するように変更されています。

CDN を利用することにはいくつかのメリットがあります。

- コンテンツ配信ネットワークによる低遅延で信頼性の高い通信
- `/.well-known/apple-app-site-association` への集中的なリクエストの削減

一方で、Apple CDN を利用することにより次のようなデメリットも出てきました。

- `/.well-known/apple-app-site-association` の変更が即時反映されない
- Apple CDN のクローラが `/.well-known/apple-app-site-association` にアクセス出来ず、キャッシュされない
  - 社内ネットワーク内での利用を前提とした従業員向けアプリ
  - 

## iOS 13以前の UniversalLinks の仕組み

:::message
既に十分に UniversalLink の仕組みを理解している方はスキップ推奨です。
:::

まず UniversalLink の仕組みをおさらいしましょう。

Universal Links を適用するにはアプリとサーバーサイドの両方に設定をする必要があります。
今回は例として、`www.example.com` というURLに Universal Links を適用します。

1. アプリの `entitlement` ファイルに Accociated Domains (applinks:) を追加

```xml
<plist>
<dict>
  <key>com.apple.developer.associated-domains</key>
  <array>
    <string>applinks:www.example.com</string>
 </array>
</dict>
</plist>
```

1. サーバーサイドに `apple-app-site-association` ファイルを配置

```json
{
  "applinks": {
    "details": [
      {
        "appIDs": ["ABCDE12345.com.example.app"],
        "components": [ { "/": "*" } ]
      }
    ]
  }
}
```

次の2つが利用可能なパスです
`https://<your-domain>/.well-known/apple-app-site-association`
`https://<your-domain>/apple-app-site-association`

これでアプリとサーバサイドの準備が整いました。

### 動作確認

アプリを端末にインストール(またはアップデート)をすると、システムがアプリの `Accociated Domains (applinks:)` の内容を見て指定のURLに `apple-app-site-association` の取得を試みます。

その様子は Proxyman などの通信デバッグプロキシアプリを使うと見ることができます。

![Proxyman](https://storage.googleapis.com/zenn-user-upload/q3s7ieuens3s10owggqycqlto8mu)

この Proxyman のログからは、最初に `https://www.wantedly.com/.well-known/apple-app-site-association` を GET して HTTPステータスコード: 404 が返ってきたので、続いて `https://www.example.com/apple-app-site-association` を GET していることがわかります。

システムは取得した `apple-app-site-association` の `"applinks": { ... }` と、アプリにバンドルされた `entitlement` ファイルの `applinks:` と符合するかを確認し、一致していればURLとアプリの関連付けが有効になります。

アプリ、またはサーバーサイドの一方だけでは機能しないのが Universal Links の特徴でもあり、Custom URL schemes に対する明確な利点にもなっています。

ここまで Universal Links の仕組みについて簡単におさらいしました。

## iOS 14からの Universal Links の仕組み

冒頭でも述べましたが、アプリインストール時の `Accociated Domains (applinks:)` の検証は `/.well-known/apple-app-site-association` ではなく Apple が管理する CDN の `https://app-site-association.cdn-apple.com` を参照するようになっています。
こちらが iOS 14 の端末にアプリをインストールしたときの Proxyman のログです。

![](https://storage.googleapis.com/zenn-user-upload/tcg7aqyrbozqvz578v15jfj5qpzc)

### iOS 14 からの注意点

## まとめ



## 引用
What's New in Universal Links https://developer.apple.com/videos/play/wwdc2020/10098
Supporting Associated Domains https://developer.apple.com/documentation/safariservices/supporting_associated_domains
Associated Domains Entitlement https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_developer_associated-domains
applinks.Details https://developer.apple.com/documentation/bundleresources/applinks/details

the Apple App Site Association CDN https://developer.apple.com/forums/thread/658454?answerId=632874022#632874022