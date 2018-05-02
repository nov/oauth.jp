---
layout: post
title: "OAuth IdP Mix-up Attack - part II"
date: 2016-01-25 15:59
comments: true
categories:
---

[OAuth IdP Mix-Up Attack とは？](/blog/2016/01/12/oauth-idp-mix-up-attack/) のつづき。

OAuth ML上で、以下のどちらを採用すべきかについての議論が収まる気配のない昨今です。

* http://tools.ietf.org/html/draft-jones-oauth-mix-up-mitigation
* http://tools.ietf.org/html/draft-sakimura-oauth-meta

そんななか、もうちょっとややこしいというか致命的なケースが出て来ました。

* [Code phishing attack on OAuth 2.0 [RFC6749] | .Nat Zone](http://nat.sakimura.org/2016/01/22/code-phishing-attack-on-oauth-2-0-rfc6749/)

"Mix-up Attack" っていうやつの話をしてたはずが、いつのまにか "Code Phishing Attack" って名前になっててもはや2つの違いがよく分かんなくなってきますが、ブログ記事中では「フィッシングメールをRP Developerに送って、Client DeveloperにToken Endpointを書き変えさせる」というパターンが例示されています。

いや、そんなんに騙されるやつおらへんやろ〜、ってのがだいたいの反応かとは思いますし、RP Developerのみなさんがちゃんと注意しれてばそれで十分な話ではあるかと思います。

が、IdP視点でいうと、アホなRPからユーザーさんのTokenとかそのRPのclient_secretとかがだだ漏れになっちゃうとIdPのレピュテーションにひびいたりするので、なかなかつらいところです。

もはやここまで騙されてる状況では、Authorization Response以外何も信用できない訳で、Authorization Endpointにiss含んでも、issに紐付いたIdP Configがstaticにhard-codeされてるなら、そのhard-codeされてるconfig自体信用できないということになります。

「IdPごとに別のredirect_uriを使う」なんてのも、もはや無意味ですね。

response_type=code+id_tokenとして、fragmentについてきたID Tokenをチェックしても、Discovery無しだとダメでしょう。

そのため、IdPが取れる対応策は以下の2つのどちらかになるでしょう。

* issを返しつつ、OAuth Discoveryをサポートする
* OAuth MetaのようにToken Endpoint URLをAuthZ Responseに含める

前者は http://tools.ietf.org/html/draft-jones-oauth-mix-up-mitigation をより厳しくしたもので、後者は http://tools.ietf.org/html/draft-sakimura-oauth-meta そのものです。

個人的にはOAuth Discoveryが必須になるとHTTP Requestが増えるので、IdP的にはあまり嬉しくないなぁと思います。

ps.

Implicit FlowにおいてResource Endpointが書き換えられていることを想定したケースでは、AuthZ ResponseにResource Endpoint(s) を含めることになって、Code Flowよりさらにややこしい話になるわけですが、それはまた別の機会に...書く...かも。