---
layout: post
title: OAuth 2.0 の code は漏れても大丈夫ってホント!?
date: 2014-05-09 09:59
comments: true
categories:
---

昨日の[Covert Redirect で Query 漏れるケースもある!?](/blog/2014/05/08/covert-redirect-and-non-30x-redirects/)や[OAuth 2.0 の脆弱性 (!?) "Covert Redirect" とは](/blog/2014/05/07/covert-redirect/)にあるように、OAuth 2.0 の code が漏れちゃうことも、ありえます。

漏れないためにやるべきことは、上記の記事や[Facebook Login で Covert Redirect を防止する](/blog/2014/05/08/facebook-login-and-covert-redirect/)なんかでも紹介してるので、そちら読んでください。

で、今回の内容は**「code が漏れたら何がまずいのか」**についてです。

### 「code は漏れても大丈夫」説

[「Covert Redirect」についての John Bradley 氏の解説（追記あり）](http://www.openid.or.jp/blog/2014/05/covert-redirect-and-its-real-impact-on-oauth-and-openid-connect.html)にも、こうありましたね。

<blockquote>
OAuth と OpenID Connect には複数の response_type があるんだけど、さきのリポートの著者は、最も一般的な response_type が "code" であることに触れず、またクライアント・クレデンシャルを使ってもう一度呼び出しを行わないとアクセス・トークンは手に入らないってことも無視してる。つまり、たしかに "code" がオープン・リダイレクターを経由して漏洩するかもしれない、という点については彼が正しい。けど、その code を使って攻撃者がなにかできるわけではないよ。これこそまさに、"code" response_type を使うことで得られる効果的な緩和策だね。
</blockquote>

あれ？code 漏れても問題なさそうですね？

ほんとでしょうか？

#### [仕様策定者視点での答え]

<blockquote>
本当です。みんながちゃんと OAuth の仕様に沿って実装していれば。仕様に沿って実装してない場合は知らん。ちゃんとやれ。
</blockquote>

#### [OAuth Server 実装者視点での答え]

<blockquote>
本当です。Client がちゃんと client secret を漏洩させずに、OAuth の仕様に沿って実装していれば。client secret 漏らしたり仕様に沿って実装してない場合は、うちのユーザーに迷惑かかるかもしれんし、あなたのアプリ停止しちゃうね。
</blockquote>

#### [OAuth Client 実装者視点での答え]

<blockquote>
え？俺ちゃんと仕様に沿って実装できてんの？

...まぁ動いてるしできてる、よね！？

それに万が一 code 漏れても、うちに被害及ぶってより OAuth Server に被害及ぶはずやし、OAuth Server がちゃんと対策してれば、まぁなんとかなるよね？
</blockquote>

**ゆとりか！(｀ヘ´#)**

OAuth Client 実装者がこういう考えだと、一瞬でやられちゃいそうですねぇ。

ってことで、ここでは**「code が漏れたら OAuth Client 側でアカウント乗っ取りが発生する」**ケースについて考えてみましょう。

<!-- more -->

### 前提条件

あなたのサイトは「Facebook ID でログイン」できるようになっています。(ちなみにここでは一番世に広まっている Facebook を挙げますが、問題の本質は Facebook に限定された話ではないので、ID Provider が GitHub や LinkedIn な場合でも同様です)

あなたのサイトからは client secret は漏れていませんし、Implicit Flow も使ってないので、「Token 置換攻撃 ※1」は受けません。

あなたのサイトには open redirector があり、被害者の code が攻撃者に漏洩しています。(or している可能性があります)

client secret は漏れていないので、攻撃者が漏洩した code を access token と交換することは、できません。

### 攻撃例

攻撃者は、あなたのサイトの正規の redirect_uri に、漏洩した code を送りつけます。

<pre>
GET /facebook/callback?code=&lt;leaked-code&gt;
Host: client.example.com
</pre>

ここであなたのサイトはこのリクエストを Facebook から正規のリダイレクトレスポンスとして処理します。通常、だいたいこんな処理を行うのではないでしょうか。

1. Facebook の Token Endpoint に code を POST し、access token を取得する。
2. access token を使って <code>GET /me</code> にアクセスし、ユーザーの Facebook UserID を取得する。
3. 取得した Facebook UserID と紐づくアカウントを特定し、local の session cookie を発行するなどしてログイン済にする。

あれ？攻撃者が漏洩した code を使って、**あなたのサイトに被害者のアカウントで**ログインできてしまいましたね？

なんか Token 置換攻撃と同じような現象が起こっちゃいましたね？

こんなことが起こりえる以上、

<blockquote>
万が一 code 漏れても、うちに被害及ぶってより OAuth Server に被害及ぶはずやし、OAuth Server がちゃんと対策してれば、まぁなんとかなるよね？
</blockquote>

なんて悠長なことは、言ってられませんね。

#### [捕捉]

なお、code <-> token 交換時には redirect_uri を Token Endpoint に送ることになるので、ここで Server 側がちゃんと code 発行時の redirect_uri とここで送られて来る redirect_uri を exact match で検証していれば、Step.1 でエラーになってこの攻撃は成立しないはずです。

実際 Facebook はこのタイミングでは exact match 必須なんで、正規の redirect_uri が open redirector になってない限りは上記の攻撃は Step.1 で失敗します。

が、残念ながら世の中には 某ithub みたいに (ry

### 対策方法

まず、この攻撃が発生してしまう根本原因は、ここです。

<blockquote>
ここであなたのサイトはこのリクエストを Facebook から正規のリダイレクトレスポンスとして処理します。
</blockquote>

これを防げれば OK です。

これを防ぐための一番簡単な方法は、**被害者と攻撃者が異なる UserAgent を使っていることを検知する**ことです。

いわゆる CSRF 対策ができてればいいんですね。

ここで @ritou 先生の[OAuth 2.0のstateとredirect_uriとOpenID ConnectのnonceとID Tokenについて](http://d.hatena.ne.jp/ritou/20121008/1349695124)でも紹介されてる state パラメータってやつが登場するわけです。

この state パラメータについては、Client が任意であらゆる文字列を指定できてしまうので、前提知識の乏しい Developer にとってはいまいち何を指定すればいいか分からないかもしれません。

@ritou 先生の記事に従って、session ID のハッシュ値を指定してもいいですし、先日 John Bradley が出してきた[こちらの OAuth 2.0 拡張仕様](http://tools.ietf.org/html/draft-bradley-jwt-encoded-oauth-state-00)を参考にしても良いでしょう。

### 結論

ちゃんと state 使ってない OAuth Client に関しては、**「code なら漏れても大丈夫」なんてウソです**。

OAuth Server には、code 置換攻撃を防ぐ手だてはありません。

OAuth Client がちゃんと実装するしかありません。

open redirector だらけで state パラメータもまともに使っていないような OAuth Client なんて、もう救いようが...(ry

まぁ、みんなちゃんとしましょう。

※1「Token 置換攻撃」については、これまでもたびたび書いてきたので、これらの記事をどうぞ。

* [@IT - RFCとなった「OAuth 2.0」――その要点は？ (2/2)](http://www.atmarkit.co.jp/ait/articles/1209/10/news105_2.html)
* ["なんちゃら iOS SDK" でありそうな被害例](/blog/2012/02/08/ios-sdk/)
* [「OAuth 2.0 (Implicit Flow) でログイン」の被害例](/blog/2012/02/07/oauth-20-implicit-flow/)
* [OAuth 2.0 Implicit Flow で認証の問題点、再び。](/blog/2012/06/29/oauth-20-implicit-flow-78852/)
