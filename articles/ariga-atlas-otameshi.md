---
title: "RDB版Terraform Arig/Atlas で次世代スキーママイグレーション"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["db","migration","schema","postgresql"]
published: false
---

# DBスキーママイグレーションツールの新星 Atlas

Atlasは
> Go言語のORM Ent を開発しているチームが作っており、同ORMを利用するとEntの生成したコードを利用した便利なデータシーディングなどが行えます
> https://entgo.io/ja/docs/versioned/intro/


Atlas Cloudを使えば、AltasファイルからのER図の生成や、CIの実行状況の確認も行えます。

# 試してみる

Dockerなどで準備してもいいのですが、今回はよりお手軽な Postgres-WASMで試してみます

以下のリンクにアクセスします

https://wasm.supabase.com/

Virtual Machine > Network > Start ボタンを押すと 、あなたのブラウザの中でPostgreSQLが起動します

画面左下に Starting Network ... と表示され、しばらくすると以下のような表示になります
```
host:proxy.wasm.supabase.com port:[Port]
psql postgres://postgres@proxy.wasm.supabase.com:[Port]
```

[PostgreSQLの接続文字列のフォーマット](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING)は以下です

```
postgresql://[user]:[password]@[host]:[port]
```


2行目の、psql の後の文字列が、DB接続文字列です
> 接続文字列とは、データソース（通常はデータベースエンジン）に関する情報およびデータソースへの接続に必要な情報が含まれる文字列を指します。


`ALTER USER postgres WITH PASSWORD 'password';` を実行して、パスワードを適当なものに変更してください
psql postgres://postgres@proxy.wasm.supabase.com:6155
psql postgres://postgres:my_password@proxy.wasm.supabase.com:<PORT>

# Altasを使って、ダウンタイムゼロでカラムの型を変えてみる

スキーママイグレーションは、適応する順序によってはサービスにダウンタイムが発生してしまいます。
今回は、カラムの型を変更するという作業をダウンタイムが発生しないように行う手順を、Atlasを用いた場合どう行うかを試していきます

基本的には、ゼロダウンタイムでこのようなスキーマ移行を行うには、移行前と移行後の両方のスキーマをサポートする「移行期」を間に挟む方法が取られます

- 新しい型のカラムを追加
- 新旧両方のカラムにデータを書き込むようにアプリを修正しデプロイ
- 古いカラムから新しいカラムにデータを移行
- 新しいカラムからデータを読み込むようにアプリを修正しデプロイ
- 最後に、古い列を削除する


```sql
UPDATE users
SET meta_json = CASE
        -- when meta is valid json stores it as is.
        WHEN JSON_VALID(cast(meta as char)) = 1 THEN cast(cast(meta as char) as json)
        -- if meta is not valid json, store it as an empty object.
        ELSE JSON_SET('{}')
    END
WHERE meta_json is null;
```

```
atlas migrate hash --dir "file://ent/mirate/migrations"
```

# 参考リンク

https://docs.gitlab.com/ee/development/database/avoiding_downtime_in_migrations.html

https://entgo.io/blog/2022/12/01/changing-column-types-with-zero-downtime

https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING