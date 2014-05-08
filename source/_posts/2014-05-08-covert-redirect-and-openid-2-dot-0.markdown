---
layout: post
title: "OpenID 2.0 における Covert Redirect と RP Discovery"
date: 2014-05-08 10:57
comments: true
categories:
---

さて、[OAuth 2.0 & OpenID Connect における Covert Redirect の問題](/blog/2014/05/07/covert-redirect-in-implicit-flow/) についてはまとめたので、次は OpenID 2.0 についてです。

いいですか、OpenID Connect ではなく、OpenID 2.0 ですよ。古い方です。懐かしいですね、[OpenID Authentication 2.0 - Final](http://openid.net/specs/openid-authentication-2_0.html)。

そういえば、OpenID Foundation Japan の翻訳 WG を立ち上げて、一番最初に翻訳したのが [OpenID Authentication 2.0 の 日本語訳](http://openid-foundation-japan.github.io/openid-authentication.html)でしたね...(遠い目

と、昔話はそれくらいにして、本題に入りましょう。

### まずは Covert Redirect 発生条件について

OpenID 2.0 では、レスポンスパラメータは query に引っ付いてきます。fragment は使いません。

なので、open redirector のリダイレクト先には、通常はレスポンスパラメータは渡りません。

全ての query パラメータをリダイレクト先に引き継いでしまうという、奇妙な open redirector を用意してる場合にのみレスポンスパラメータが外部に漏洩します。

まずはこの時点でそんなに心配することはなさそうですね。

まぁでもせっかくなんで、話続けますね。

<!-- more -->

### RP Discovery ってのがあってだな

さて、OpenID 2.0 には、[RP Discovery というものが定義されていた](http://openid.net/specs/openid-authentication-2_0.html#rp_discovery) ([翻訳版はこちら](http://openid-foundation-japan.github.io/openid-authentication.html#rp_discovery)) のを、おぼろげに覚えている方もおられるのではないでしょうか？

え、そんなの聞いた事が無い？でしょうね！w

OpenID 2.0 は OAuth とちがって、通常 RP が OP に対して事前登録を行うことはありません。

* RP: Relying Party, OAuth 2.0 の Client 相当
* OP: OpenID Provider, OAuth 2.0 の Server 相当

OP と RP の間では、動的に association というものを確立します。association って何？って人には、「最初に Temporary な共有秘密鍵をする」って言っちゃえばいいですかね。

association が確立されると、RP は各種リクエストパラメータと共に return_to (OAuth 2.0 の redirect_uri 相当) ってのを指定して、ユーザーを OP にリダイレクトさせます。

で、OP はユーザー認証とか同意取得とか OAuth と似たようなことしてから、最終的に各種レスポンスパラメータを添えてユーザーを指定された RP の return_to にリダイレクトして戻します。

で、ここまでが全部 dynamic なんで、事前登録された return_to との完全一致、なんてことがそもそもありえないんです。

### RP Discovery ってのがあってだな (part 2)

まぁ既に RP Discovery についての記述を読んだみなさんには釈迦に説法、ネコに小判ですか？あれですね、日本語難しいですね。

まぁあれです、OpenID 2.0 の仕様だけで RP Discovery 理解しろとかムリゲーな気がします。Yadis ってのがあってだな...いや、やめよう。

実際に、これまた懐かしの iKnow! を例に、RP Discovery ってのを概観してみましょう。

まず iKnow! に OpenID でログインする際のリクエストパラメータを眺めると、"openid.realm=https://*.iknow.jp/open_ids" ってのが含まれてるのが分かるかと思います。

iKnow! はちょっとレアケースで、サブドメインを許可する為に "*.iknow.jp" になってますが、そういう場合は "\*" を "www" に置き換えて "https://www.iknow.jp/open_ids" にアクセスします。

<script src="https://gist.github.com/nov/c702782baa98c75702dc.js"></script>

あ、リダイレクトしましたね。これは follow します。

<script src="https://gist.github.com/nov/67c491166bed6f9cdb7e.js"></script>

で、このレスポンスヘッダに注目です。

<pre>X-XRDS-Location: https://iknow.jp/discovery.xrds</pre>

XRDS ってのがありますね。これが、RP の metadata が記述されたドキュメントです。

で、今度はこの XRDS endpoint にアクセスします。

<script src="https://gist.github.com/nov/b6173c3195fa49e1c668.js"></script>

はい、return_to が指定されてますね。

<pre>&lt;Service priority="0"&gt;
  &lt;Type>http://specs.openid.net/auth/2.0/return_to&lt;/Type&gt;
  &lt;URI>https://iknow.jp/open_ids?_method=GET&lt;/URI&gt;
&lt;/Service&gt;</pre>

で、RP Discovery をサポートしてる OP は、RP の realm にアクセスしてみて、レスポンスヘッダに "X-XRDS-Location" ってのがあれば、return_to の検証を行う、ってのが一般的な実装かと思います。

### RP Discovery って必要なの？

大概の OP では不要です。

てか RP 側の人たちはこれほとんど知らんかったんじゃないかと。

噂によると一部 OP では RP Discovery が必須なやつもいるらしいですが、Gooogle, Yahoo! Japan, mixi はじめ、ほぼ全ての主要 OP では RP Discovery は OPTIONAL です。

また XRDS 中の return_to に指定されている URL とリクエストパラメータに含まれる return_to のマッチングルールも、完全一致だったり部分一致だったりは OP ごとに違うようです。

さらには RP 側が "X-XRDS-Location" を返さない限り、OP 側では RP Discovery をスキップします。

よって、RP Discovery を実装していない RP に open redirector が存在する場合は、それが悪用されるケースはあり得ます。(まぁこの記事の先頭で紹介した発生条件を満たす open redirector が存在するケースは稀ですが)

### 対策

まず query を全部引き継いでリダイレクトするのやめましょう。いや、そもそもみなさんはそんなことしてないと思いますけど。

RP Discovery は、まぁ頑張ってやってみてもいいかもしれないけど、結構複雑なのであまり気軽に実装できる気はしません。

...懐かしかったですね。