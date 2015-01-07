---
layout: post
title: "Covert Redirect で Query 漏れるケースもある!?"
date: 2014-05-08 15:47
comments: true
categories:
---

Covert Redirect 連載最後は、location.href とか &lt;meta http-equiv="Refresh"&gt; とかだと referrer 通じて query に含まれる code とかも漏れるかもね！ってお話です。

古いサイトですが、こことか見れば大体いいんじゃないでしょうか => [リファラ実験](http://www.teria.com/~koseki/memo/referrer/)

で、こういうリダイレクトをしてる箇所が OAuth 2.0 の redirect_uri なり OpenID 2.0 の return_to なりに指定されていれば、query に付いた code なり email なりがリファラ経由で外部に漏れるよね、という。

はい、漏れますね！ (投げやり

さて、そんなに該当例多くはないと思うのですが、ここまで該当してしまったサービスは、どうしますかねぇ...

Facebook であれば、[Facebook Login で Covert Redirect を防止する](/blog/2014/05/08/facebook-login-and-covert-redirect/)にあるような方法で回避できますが、それ以外の OAuth Provider なり OpenID 2.0 Provider と連携してる場合は、困っちゃいますねぇ...

<!-- more -->

### 被害例

[OAuth 2.0 の code が漏れた場合について](/blog/2014/05/07/covert-redirect/)は既に書きました。

Client が redirect_uri 上で state パラメータのチェックを怠っていれば、code 置換攻撃が可能になるっていうアレです。

OpenID 2.0 のレスポンスパラメータが漏れた場合は、そのパラメータに email や name なんかが含まれてる場合があるので、そういった場合はそれらが外部に漏洩します。

また世の中には OpenID 2.0 の nonce を OAuth 2.0 の access token のように使って API アクセスさせる事業者があったりするんで、そういった場合は acccess token 相当のものが漏洩します。

それら access token 相当の nonce が漏れた場合の実害については、提供される API に依存するのでここでは未知です。

### 対策方法

リダイレクト方法変えろ、って以外の回避策を思いついたら、この下に書きます。

