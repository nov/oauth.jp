---
layout: post
title: Implicit Flow では Covert Redirect で Token 漏れるね
date: 2014-05-07 18:41
comments: true
categories:
---

この記事は、[先ほど書いたこちらの記事](/blog/2014/05/07/covert-redirect)の訂正版です。

**記事に入る前に、まずは全シンガポールにお詫び申し上げますm_ _m**

さて、Covert Redirect についての説明は...超絶取り消し線はいりまくってる前の記事を読んでください、でいいでしょうか？

で、訂正分だけ以下に。

### Fragment Handling in Redirect

[宮川さんが記事にしてますね](http://weblog.bulknews.net/post/85008516879/covert-redirect-vulnerability-with-oauth-2)。

英語だけど。

で、まぁ要するに、(Modern Browser は) 30x リダイレクト時に**リダイレクト元に付いてた URL fragment をリダイレクト先にも引っ付ける**、と。

fragment は server-side には送られないけど、クライアントサイドではリダイレクト先に引き継がれる、と。

試しに http://id.gree.net/#foobar にアクセスすると、(未ログイン時は) リダイレクトを繰り返して http://games.gree.net/welcome#foobar にリダイレクトされるかと思います。

GREE のサーバーには **#foobar** の部分は送られませんが、games.gree.net/welcome という **Endpoint 上の JS からは、#foobar** にアクセスできます。

なので、Covert Redirect のケースでも、open redirector をつかって最終的に被害者がリダイレクトしてくる endpoint に攻撃者が JS を仕込んでそれを自分のサーバーにでも送るようにしておけば、**access token が漏洩します**。

もちろん[先ほどの記事にあるように](/blog/2014/05/07/covert-redirect)、Authorization Code が漏洩するケースもありますが、open redirector の実装詳細に依存しない分、Implicit Flow において fragment に含まれる access token が漏洩する方が可能性としては高いでしょう。

<!-- more -->

### 仕様のバグではないが...

[宮川さんが記事](http://weblog.bulknews.net/post/85008516879/covert-redirect-vulnerability-with-oauth-2)にもあるように、[RFC6819 - OAuth 2.0 Threat Model and Security Considerations](http://tools.ietf.org/html/rfc6819) の Section 4.2.4, 5.2.3.5 あたりにはこの問題が指摘されています。([RFC6819 翻訳版はこちら](http://openid-foundation-japan.github.io/rfc6819.ja.html#implicit_flow))

OpenID Connect では、**そもそも redirect_uri の部分一致を認めていません**。([Core](http://openid.net/specs/openid-connect-core-1_0.html) Section 3.2.2.1 参照, [日本語版はこちら](http://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html))

OAuth 1.0 では Refresh Token Secret が漏れない限り Access Token が Covert Redirect 経由で漏洩することは無いですし、OpenID 2.0 でも RP Discovery というのをきちんとやっていれば、redirect_uri の exact match 相当のことができます。

なので、特に Covert Redirect がこれらの仕様のバグであるということではありません。

が、まずい実装は結構ある、というのが、[元記事](http://tetraph.com/covert_redirect/oauth2_openid_covert_redirect.html)の Provider List が伝えたかったことなのでしょう。

まぁ確かに redirect_uri の部分一致認めてる OAuth Server なんてそこら中にあるし、OpenID 2.0 に至っては RP Discovery 必須な OP ってどこよ？、みたいな状態ですし、OpenID Connect 実装って言っても完全に OpenID Connect の仕様に準拠してるかどうかあやしいですし...

いや、Google の Connect 実装は exact match ですよ。でも元記事見る限り、Google のは OpenID 2.0 実装が問題にされてますね。

Y!J の OpenID Connect も大丈夫、なんじゃないかな。これは Y!J Developers Blog に期待するとして、Y!J も OpenID 2.0 実装の方は...あやしいな...

他の OpenID Connect 実装は...元々 OAuth 2.0 実装済やったとこが後から OpenID Connect 対応したよ！とか言ってきたケースとかは...あやしいかもしれませんね...

他に何があったっけ...

### 攻撃パターン

攻撃者は、redirect_uri を Client 上に存在する open redirector の endpoint に書き換えたリンクを被害者に踏ませます。

このリンクを踏ませる方法は何でもいいのですが、Client 側に悪意が無い限り、基本的には「通常のフロー」とは異なる方法でリンクを踏ませることになるでしょう。フィッシングですね。

で、あとは被害者が同意ボタンを押せば、open redirector 経由で攻撃者が指定した URL に、(ユーザーが Modern Browser を利用している場合は) fragment に access token を含んだ状態で、被害者がアクセスしてきます。

あとはその fragment から JS をつかって access token を抜き出して...ってのはこの記事冒頭でも書きましたね。

### 対策

#### Server 側ができること

redirect_uri の完全一致を必須にする。

いままで動いてた既存アプリが突然動かなくなるケースもあるでしょうし、そうそう気軽に変更できる部分では無いかもしれませんが、ある程度の猶予期間を用意した上で、redirect_uri の完全一致を必須にするのが良いでしょう。

あと、そもそもこの問題が自社の各プロトコル実装で置きうるのかどうか、redirect_uri の検証方法をいま一度 Developer 向けに広報する、とかはやった方がいいかもしれませんね。

#### Client 側ができること

例えば Facebook は、redirect_uri の完全一致を行うようにアプリ設定画面で指定することができます。(Advanced Settings の "Valid OAuth redirect URIs" ってやつ)

"Valid OAuth redirect URIs" はデフォルトでは空っぽで、その状態では redirect_uri の部分一致が許容されますが、ここに URL を指定することでFB 側に redirect_uri の完全一致を要求することができます。

そもそも Implicit Flow なんてつかわねーよ、って場合は、[appsecret_proof](https://developers.facebook.com/docs/graph-api/securing-requests/) なんていう FB の独自拡張を利用することもできそうですが、まぁこれはそのうち調べようと心に誓いつつ全く調べてないのでよく知らないです。

あと open redirector 作らない、ってのね。

こらそこ、「あっ、はい」とか言わない。

#### User 側ができること

フィッシングに引っかかるな、くらいしか思いつかないですが、まぁそれじゃ引っかかる時は引っかかるでしょうね...

Client と Server が頑張る、のを応援する、ってのが一番いいかもしれませんね。

### 最後に

改めまして、全シンガポールにお詫び申し上げますm_ _m

あと Twitter で mention いただいた [@miyagawa](https://twitter.com/miyagawa) さんと、LINE の某厨部屋で連絡くれた [@mad_p](https://twitter.com/mad_p) さん、ありがとうございますm_ _m