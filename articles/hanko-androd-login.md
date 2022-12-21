---
title: "Hankoを使ってAndroidでpassKeyログイン試してみた"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Android"]
published: true
---

本記事は「株式会社RetailAI X Advent Calendar 2022」の17日目の記事です。
昨日は@takudooonさんの「[ESP32を通してUSBデバイスへの理解をちょっと深めた話](https://zenn.dev/takudooon/articles/79345e925bba01)」でした。

---

パスワードレスな認証を実現する認証ミドルウェア Hankoを、モバイルアプリから利用する手順とサンプルコードが公式から提供されたので、勉強のために動かしてみます。

https://docs.hanko.io/guides/mobile_guide

# 試してみる

Hanko CloudでHankoプロジェクトを作成（セルフホストもできます）

https://cloud.hanko.io


サンプルアプリのソースコードをClone

```
git clone https://github.com/teamhanko/hanko-android-example.git
```


`app/src/main/java/io/hanko/hanko_mobile_example/KtorClient.kt`の`BASE_URL`変数の値を、作成したHankoバックエンドのURLに変更します

> プロジェクトの　Sttings > Hanko API URL　からコピー

```kotlin
object ApiRoutes {
    private const val BASE_URL = "<YOUR_HANKO_API>"
```

これだけでも、ワンタイムパスワードによるユーザー登録とログインができます

このままではパスキーは登録できないので、アプリリンクを設定します。

## アプリリンク設定

https://developers.google.com/identity/fido/android/native-apps

以下のような内容の`assetlinks.json`を用意して、ホスティングします。
> レスポンスヘッダーは `Content-Type: application/json`

- `package_name`に開発中のアプリのパッケージ名 (今回の場合、`io.hanko.hanko_mobile_example`)

- `sha256_cert_fingerprints`には、アプリの署名証明書の SHA256 フィンガープリントを記入します。
  - > `keytool -list -v -keystore ~/.android/debug.keystore`　で取得できます

```json
[
  {
  "relation": ["delegate_permission/common.handle_all_urls", "delegate_permission/common.get_login_creds"],
  "target": {
    "namespace": "android_app",
    "package_name": "パッケージ名",
    "sha256_cert_fingerprints": ["DE:AD:BE:EF:..."]
    }
  }
]
```

自分は、ローカルにサーバーを起動して[Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/)で外部公開しました。

`app/src/main/res/values/strings.xml`ファイルで指定されている`assetlinks.json`のドメインを、自分がホスティングしているドメインに置き換えます。

> 例:  \"include\":"https://{project}.trycloudflare.com/.well-known/assetlinks.json\"


# hanko cloud側の設定

プロジェクト Sttings > Hanko APP URL に assetlinks.jsonをホスティングしているドメインを入力

プロジェクト Sttings > Android APK key hash にAndroid APK キー ハッシュを入力

入力する値は以下のコマンドで取得できます
```
echo 'android:apk-key-hash:'"$(keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha256 -binary | openssl base64 | tr -- '+/' '-_' | tr -d '=')"
```


# 起動

起動する前に、動かす端末やエミュレーターで以下を行ってください

- Googleアカウントにログイン
- スクリーンロックを設定

アプリを起動すると以下の機能が利用できました

- メールアドレスとワンタイムパスワードでログイン
- パスキーの登録
- パスキーでログイン

# 感想


パスワードレスの認証を導入は、店舗オペレーションの負担軽減とセキュリティの強化につながると考えています。

今回、勉強のためにHankoを利用して、passKeyバックエンドがすぐに用意でき、大変便利でした。

次は、iOSでのpassKey認証を試そうと思います。

# 参考

https://docs.hanko.io/guides/mobile_guide

https://codelabs.developers.google.com/codelabs/fido2-for-android#0

https://github.com/mojaloop/contrib-fido2-flutter-lib


# 次は…

明日は、@yoshitake_tatsuhiroさんの記事になります！


