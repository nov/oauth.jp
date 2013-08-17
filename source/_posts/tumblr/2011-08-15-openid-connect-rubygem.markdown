---
layout: post
title: OpenID Connect RubyGem リリース
tags:
- JustMigrate
---

OpenID Connect の OP & RP 用の RubyGem をリリースしました。

https://github.com/nov/openid_connect

同時にサンプル OP サイトも公開しました。

サイト: https://openid-connect.herokuapp.com/

ソース: https://github.com/nov/openid_connect_sample

サンプルサイトに Facebook / Google ID でログインすると、まず OAuth Client の登録を要求されます。
OAuth Client を登録すると、以下の様に各 response_type ごとの Authorization Flow を開始するボタンが表示されます。

とりあえず OpenID Connect の UserInfo Endpoint にアクセスしたい場合は、token を選ぶのが一番手っ取り早いでしょう。
Access Token を取得したら、以下のように UserInfo Endpoint にアクセスしてみてください。

https://openid-connect.herokuapp.com/user_info?access_token=YOUR_TOKEN

こんな感じで JSON レスポンスが返ってくるはずです。

仕様の詳細はまだ把握しきれてないけど、とりあえず OpenID TechNight in Kansai にサンプル間に合った！
今後も 12/1 の OpenID Summit in Tokyo に向けて、徐々に機能追加していきます。
pull request 大歓迎♪