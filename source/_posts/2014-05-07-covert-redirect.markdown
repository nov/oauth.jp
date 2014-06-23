---
layout: post
title: OAuth 2.0 の脆弱性 (!?) "Covert Redirect" とは
date: 2014-05-07 13:53
comments: true
categories:
---

### 訂正

リダイレクト時の fragment の扱いを勘違いしていたため、本記事全体訂正します。

細かく訂正いれてると分けわかんなくなってきたんで、[新しい記事書きました](/blog/2014/05/07/covert-redirect-in-implicit-flow/)。

ゴールデンウィークまっただなかに Twitter で海外の ID 厨から袋だたきにあってたので、もうこの問題は片付いただろうとすっかり油断してた「Covert Redirect」の件ですが、日本でもゴールデンウィーク明けてバズりだしたので、一旦問題を整理した方がよさそうですね。

### 事の発端

Wang Jing さんていうシンガポールの大学院生が、[こんなサイト](http://tetraph.com/covert_redirect/oauth2_openid_covert_redirect.html)を公開すると共に [CNet](http://www.cnet.com/news/serious-security-flaw-in-oauth-and-openid-discovered/) はじめ各種メディアが取り上げたのが、バズりだした発端のようです。

### 前提知識

OAuth 2.0 や OpenID Connect だけでなく、OAuth 1.0 や OpenID 1.0/2.0 や SAML なんかでも、**2つのサービスの間でリダイレクトを経由して**、Access Token や ID Token などのいわゆる **"Security Token" の授受** が行われます。

Covert Redirect の問題は本質的にはこれら全てに影響しますが、各仕様一個一個見て行くといろいろ疲れるので、ここでは OAuth 2.0 に絞って話をすすめましょう。

OAuth 2.0 では、まず Client が End-User の UserAgent を Server の Authorization Endpoint というところにリダイレクトします。よく Twitter とか Facebook の ID で外部サービスにログインするとき、一度 Twitter や Facebook にリダイレクトして、同意画面表示されますよね？あれです。

![FB の同意画面](https://fbcdn-dragon-a.akamaihd.net/hphotos-ak-prn1/t39.2178-6/851557_535801936465660_169463870_n.png)

で、みなさん何も考えず熟慮の結果、同意ボタンを押すじゃないですか。そして、元のサイト (= Client) に戻って来る、と。

OAuth 愛好家の間で "OAuth Dance" なんて呼ばれてるアレですね。

で、今回の話は、その **"OAuth Dance" 中の話** です。

<!-- more -->

### Covert Redirect てなんだ？

Server 側の同意画面で同意ボタンを押すと、Client にリダイレクトして戻って来る。

そう、同意すると、元の Client に戻って来るんですよ。

で、Client 側に open redirector があって、まさにその open redirector めがけて戻ってきちゃったりしたら、そらもう open に redirect する訳です。

さらに悪いことに、open redirector の実装によっては、リダイレクト時に query に引っ付いてきたデータがそのリダイレクト先にまで渡っちゃうこともあるんです。

渡された query パラメータを全てリダイレクト先にまで forward するような実装、ちょっと正気の沙汰とは思えないかもしれないけど、まぁそういうのは実際あるらしいですし。(ESPN ってサイトがそうらしい)

こうして **Client 側の open redirector が悪用** されて、**query についた Security Token が攻撃者に漏れちゃう** っていうこれ、これが **Covert Redirect** なんです。

### Redirect URI に open redirector !?

え？なんで redirect_uri に指定するエンドポイントに open redirector が存在すんだよ！って？

えぇ、普通ありえないですよね。

いくら open redirector だらけのザルサイトでも、さすがに redirect_uri に指定するエンドポイントくらいは...

*...いや、P言語ならありえなくもない気がしてきましたけど...*

でも Covert Redirect で想定されているケースでは、Client が本来指定する redirect_uri とは別の URL が、攻撃者によって redirect_uri に指定されるというのを想定してます。

で、Server 側では、事前に登録済の redirect_uri と部分一致すれば OK にしちゃうケースが結構あるんです。(domain だけが一致すれば OK とか)

この、**Server 側の甘めの redirect_uri 検証** と、**Client 側が内包する open redirector**、Covert Redirect はこの2つがうまいことくみ合わさって成立するんです。

Server 側では、一度部分一致を許した実装が広く世の中につかわれだしてしまうと、そこの仕様を変更するのはなかなか大変なんで、いざ直そうと思っても数ヶ月から数年の政治的調整作業等が必要になっちゃうかもですね。

### で、どれくらいヤバいの？

<s>実際の影響については、OpenID Connect の仕様策定者の一人でもある [John Bradley さんが記事書いてる](http://www.thread-safe.com/2014/05/covert-redirect-and-its-real-impact-on.html)ので、まぁそれを読んでください。

ほんとはちゃんとこれ訳そうと思ったんですが、[OpenID Foundation Japan から翻訳版](http://www.openid.or.jp/blog/2014/05/covert-redirect-and-its-real-impact-on-oauth-and-openid-connect.html)まで出たので、もういんじゃね？って空気を感じています。

*もういんじゃね？って空気を感じながらも、必死に記事を書いてる僕の心境、お察しください...*

で、まぁそんな僕の気持ちなんてスルーして翻訳版なり原文なりを要約すると、Covert Redirect によって **access token 漏れるとか、そんな心配は無い**ってことです。

さらに言えば、**それ別に新しい発見でも何でも無いし**って。

*え？？漏れないの？？発見でもないの？？*

*シンガポールウソばっかなの？？*</s>

### ちょっとヤバいかも (?) なケース

ただ、Authorization Code が漏れることはあり得て、被害者の code を攻撃者が正規 Client に送りつけて来て、攻撃者の Client 上のアカウントと被害者の Server 上の access token が紐づいちゃう、なんてことはあり得なくはないですね。

FB Login つかってるサイトでは、Client 上の既に FB と Connect 済の被害者のアカウントに、攻撃者がログインできちゃうかもしれないですね。

ただこれは、各 Client が state パラメーターってのをつかって CSRF 対策することで防ぐことができます。

state パラメータってのは...まぁ @ritou 先生のブログを読んでみてください。

http://d.hatena.ne.jp/ritou/20121008/1349695124

### 結論

<s>Covert Redirect によって **access token 漏れるとか、そんな心配は無い**。</s>

ただし悪用される open redirector の実装によっては、**code は漏れることもあり得る**。

code が漏れて CSRF に脆弱な Client と組合わさると、被害者の code が本来紐づくべきでない攻撃者のアカウントと紐づいて、Client 側でアカウントが乗っ取られたりってことはあり得る。

### 対策

これから Server 作るときは、redirect_uri は事前登録必須 + 完全一致にしようね。

(特に外部 ID ログイン目的で) OAuth 2.0 を利用する Client 作ってる人は、state パラメータちゃんとチェックしようね。

以上、仕事に戻ります (>_<)>

*ほんとゴールデンウィーク明け初日から...*