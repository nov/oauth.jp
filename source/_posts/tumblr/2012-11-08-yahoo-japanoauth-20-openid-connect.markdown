---
layout: post
title: Yahoo! JapanがOAuth 2.0 & OpenID Connectに対応
tags:
- JustMigrate
---
<p>2011年12月のOpenID Summit Tokyoで、2012年中のOpenID Connect対応を宣言したYahoo! Japanが、本日ついに宣言通りOpenID Connectをサポート開始しました。</p>
<p>もともとOAuth 2.0も対応していなかった（よね？）ので、OAuth 2.0対応も同時リリースです。</p>
<p>まだバグとかあるっぽいけど、何はともあれ世界の大手IdPの中で一番最初にproduction環境でOpenID Connect対応できたのはすばらしい！</p>
<p><iframe scrolling="no" margin src="http://www.slideshare.net/slideshow/embed_code/10433061" frameborder="0"> </iframe></p>
<div style="margin-bottom: 5px;"><strong> <a href="http://www.slideshare.net/tzmtk/openid-summit-tokyo-2011" title="OpenID Summit Tokyo 2011" target="_blank">OpenID Summit Tokyo 2011</a> </strong> from <strong><a href="http://www.slideshare.net/tzmtk" target="_blank">Taizo Matsuoka</a></strong></div>
<p>まだOpenID ConnectのDiscoveryとDynamic Registrationには対応していないので、<a href="https://connect-rp.heroku.com">Nov RP</a>に &#8220;yahoo.co.jp&#8221; とか入力しても使えない状態ですが、それは今後に期待です。</p>
<p>YConnectをちょこっと触ってみて思った要望とかは以下のgistにまとめていってます。</p>
<script src="https://gist.github.com/4031074.js?file=gistfile1.textile"></script>
<p>scopeにopenidが指定されてるのにtoken responseにid_tokenが含まれてないとか、response_type=code+id_tokenの時にcodeとid_tokenがredirect_uriのfragmentではなくqueryについているなどの問題は、ただのバグとして修正すれば良いでしょう。</p>
<p>それ以外でいま気になってる点としては、以下の2点です。</p>
<ul><li>response_type=code+tokenサポートしてもらわないとserver-side componentを持つmobile appから使おうとしたときにいろいろめんどくさいことになっちゃうのはFBとか見てたら明らですが、現状のYConnectは「サーバーサイド」アプリとして登録した場合response_typeにcodeかcode+id_tokenしか指定できないみたいです。これはちょっとやめた方が良い気がします。</li>
<li>あと、id_tokenのsignature algorithmはdefault RS256ということで進んでいるので、YConnectだけがHS256だとまたYAuthとか言われちゃったりしそうですね。</li>
</ul><p>それ以外にも、なぜかyahoo.co.jp以外のドメインのメアドはverfiedじゃなくても返してくれる (試しに <a href="mailto:hoge@hoge.com">hoge@hoge.com</a> というメアドを登録してみたらそれ返してきたw）のにyahoo.co.jpのメアドは返してくれないというのも不思議といえば不思議ですが、まぁ社内調整とかいろいろめんどくさいことありそうですし、それが原因でOpenID Connect対応のスケジュールが遅れるくらいならとりあえずそれでもいいのかなとも思います。</p>
<p>Implicitの方でscopeにopenidが指定された時にnonceが必須になってるかとかもチェックしたかったのですが、なんかいま「クライアントサイド」のアプリを登録しようとするとエラーになっちゃうバグがあるようなので、そちらはまた明日にでも。覚えてれば。</p>
<p>OAuth 2.0 draft 0の時点で対応してきたFacebookのように、これからOpenID Connectのbreaking changesに追随するのは大変だと思いますが、引き続きがんばってください！あと、OpenID Connectにbreaking changeがある時に一番大きな声で文句言える立場にいるので、ぜひOpenID Connect Interopにも積極的に参加していただければ :)</p>
