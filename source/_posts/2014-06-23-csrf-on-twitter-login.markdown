---
layout: post
title: Twitter Login にも CSRF 脆弱性ができやすい罠が!?
date: 2014-06-23 10:46
comments: true
categories:
---

OAuth 2.0 では state パラメータってのがあって、それをちゃんと使わないと CSRF 脆弱性ができちゃうよって話は、[@ritou 先生のスライド](http://www.slideshare.net/ritou/idcon17-oauth2-csrfprotectionritou)などでみなさん勉強したんではないでしょうか。state パラメータは RFC 6749 では RECOMMENDED 扱いで、REQUIRED ではありませんが、OAuth 2.0 をログインに使う場合は REQUIRED にすべきでしょう。OAuth 2.0 をログインに使うの、Token 置換攻撃とか Covert Redirect + Code 置換攻撃とか、いろんな罠がありますねぇ〜。

OAuth 1.0 ならそんなことないのに...

そう思ってた時期が、僕にもありました。

でも @ritou 先生よく言ってるじゃないですか。「Twitter の OAuth 実装クソや」って。でね、ほんとにクソやったんすよ、コレが。

<!-- more -->

さて、Developer の皆様におかれましては、もうずいぶん「Twitter ID でログイン」の実装なんて放置してて詳細忘れ去ってるころかと思いますので、まずは OAuth 1.0 の Access Token 取得までのフローを復習しましょう。

![OAuth 1.0 Flow](/images/posts/oauth1_flow.png)

思い出しましたか？

思い出せない方には[仕様を読み直していただく](http://openid-foundation-japan.github.io/rfc5849.ja.html)として、先に進みましょう。

OAuth 1.0 には Request Token + Request Token Secret というのがありました。RFC 5849 では Temporary Credentials とか言われたりもしますが、Request Token という呼び名の方が一般的でしょう。

で、この Request Token Secret、Access Token 取得時のリクエストに署名するために必要になります。「Consumer Secret と Request Token Secret の2つを連結させたものを HMAC 鍵として利用する」って、習いましたよね？

で、一般的にはこの Request Token Secret を Session と紐づけて保存すると思うんですが、これが認可レスポンスが返される箇所 (Callback URL) への CSRF 対策として作用していたんです。

認可リクエストを送った Browser と認可レスポンスを返してきた Browser が異なれば、当然認可レスポンスを返してきたブラウザでは Session と紐づく Request Token Secret が取得できないので、Access Token が取得できない。よって CSRF 対策ができている。

**が、ここで罠があるんです。**

実は Twitter は、Access Token 取得時のリクエストの署名に、**Request Token Secret を使わなくてもいい**んです。なので Session に紐づいた Request Token Secret が空の場合にそれをそのまま空文字列として処理してしまう RP は、**正常に Access Token を受け取れてしまいます**。

つまり、攻撃者が被害者にこんな URL を踏ませると、被害者は攻撃者の Twitter アカウントを使ってあなたのサイトにログインできてしまう可能性があるのです。

<pre>
https://client.example.com/callback?
  oauth_token=&lt;request-token-authorized-by-attacker&gt;&
  oauth_verifier=&lt;valid-oauth-verifier&gt;
</pre>

このパターン、すでにいくつかの「Twitter ID でログイン」を実装してるサイトでは確認してます。

この脆弱性によるリスクは [@ritou 先生のスライド](http://www.slideshare.net/ritou/idcon17-oauth2-csrfprotectionritou) に書いてあるのでそちらにゆずります。

Twitter が Request Token Secret を必須にしてないことは以下のコードで確認できます。
<script src="https://gist.github.com/nov/5e99d1999bea0e16bb74.js"></script>

ちなみに Twitter 側はこのバグは認識してるらしいのですが、直すと多くのサイトが動かなくなるのも把握しているのか、修正できないでいるようで、みなさんが個別に対策するしかないわけです。

とりあえず **Session に紐づいた Request Token Secret が無かったら、エラーにしちゃえば OK** です。

そもそも Request Token Secret が Session に紐づいてなかったり、Request Token Secret 一切使ってないなんて人は、アウトです。まぁいまから再び Twitter ID でログインとか自分で実装するよりは、人気のライブラリ探した方が良いでしょう。Rails だといまでも omniauth-twitter とかが人気なんでしょうか。

ちなみに omniauth-twitter (というか omniauth-oauth) は Callback URL で Session に Request Token Secret が入ってなければエラーになるので Safe です。Gunosy とか Doorkeeper は omniauth-twitter 使ってるのか、Safe でした。

そういや最近[日本ツイッター学会](https://twitter.com/TSJ2010)って何してんすかね？