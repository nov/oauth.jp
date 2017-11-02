---
layout: post
title: "LINE ID Login"
date: 2017-09-28 22:52
comments: true
categories:
---

LINE が OpenID Connect サポートしたみたいですね。

なんか前からしてんだと思ってたんですが、まぁいいや。

ということで、早速触ってみました。

こちらに[今回使った Ruby のサンプルスクリプト](https://gist.github.com/nov/58912dda45a65768d8f05343225780b2)置いておきます。

まぁ、ちょっと特殊な点がいくつかありますが、十分 OpenID Connect です。

気になった点は以下の通り。

<!-- more -->

### ID Token の署名アルゴリズムが HS256 (HMAC-SHA256)

HMAC ってことは、署名検証するために client_secret が必要ということです。

Native App で検証しようとすると、Native App に client_secret を埋め込まなければならない訳ですが、それはありえないので、要するに Native App で ID Token の署名検証はできないということです。

まぁ別に Native App で ID Token の署名検証するケースなんてそうそう無いとは思いますが。

### scope=openid だけでは動かない

LINE の User ID さえもらえればいいよってケースでも、Display Name とか Profile Picture とか取らないといけないっぽいです。

ちょっと OpenID Connect の思想からはずれていますね。

### state が必須

まぁ仕様的には OPTIONAL なんですけど、これはセキュリティ上はいいことなのではないでしょうか。

### Client 認証に Basic Auth は使えない

これじゃ OpenID Certification は通らないんですね。

UserInfo API もまだ無いようなんで、Certification 取得とかはもう少し先の話でしょうか。

### Access Token は JWS で署名された非 JWT で Lifetime は1ヶ月

ちょっと長く無いすか？裏に独自の Revoke の仕組みがあるんなら別にいいんですけど...

Refresh Token 発行しておいて、Access Token の有効期限が1ヶ月ってのは、結構違和感あります。


*...で、こういうの書いとくと、「LINE API Expert」ってやつになって有料スタンプ使い放題になったりするんでしょうか？*