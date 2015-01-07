---
layout: post
title: "Google OpenID 2.0 のサポート終了、OpenID Connect への移行はお早めに"
date: 2015-01-07 13:20
comments: true
categories:
---

今朝こちらの Tweet みて気がついたんですが、Google が OpenID 2.0 のサポートを2015年4月20日で終了するようです。

<blockquote class="twitter-tweet" lang="en">
	<p>Developers beware! Google has published its timeline for deprecation of OpenID 2.0:<a href="http://t.co/84cIS0wR1D">http://t.co/84cIS0wR1D</a></p>
	&mdash; Gluu (@GluuFederation)
	<a href="https://twitter.com/GluuFederation/status/552615928014577665">January 7, 2015</a>
</blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

本当にそんな短期間で OpenID 2.0 止めて大丈夫なのかって気がしますが、[Google OpenID 2.0 Shutdown Timetable](https://developers.google.com/accounts/docs/OpenID#shutdown-timetable) によると、4/20で全てが止まるようです。

僕が把握しているところでは、以下のサイトは現時点でまだ Google OpenID 2.0 を使っています。他にもいっぱいあるんでしょうが、僕は把握してません。他に知ってるのあれば教えてください。

* [iKnow!](http://iknow.jp)
* [ChatWork](http://chatwork.com)
* [ATND](http://atnd.org)

OpenID 2.0 停止により、上記のようなサイトでは、いままで Google Account でログインしてきたユーザー達はログインできなくなります。

これらのサイトは、4/20までに OpenID Connect (Google+ Signin) への移行が必要です。

<!--more-->

移行方法は [Google OpenID 2.0 Migration](https://developers.google.com/accounts/docs/OpenID) にあります。

なんだかユースケースごとに4通りの移行方法が紹介されてますが、基本は [Migrating to Google+ Sign-In](https://developers.google.com/accounts/docs/OpenID#update-to-plus) に従えば良さそうです。

細かいステップは [Migrate from OpenID 2.0 or OpenID+OAuth hybrid to Google+ Sign-In](https://developers.google.com/+/api/auth-migration#oid2) を読んでいただくとして、大雑把な Step は以下の通りです。

1. Authorization Request に OpenID 2.0 時代に使っていた realm を含める。
2. ID Token に OpenID 2.0 時代の Claimed Identifier が "openid_id" という名前で含まれて帰って来る。
3. "openid_id" をキーに既存ユーザーを探して、該当ユーザーの識別子を "sub" に置換する。

あと、Attribute Exchange 使ってプロフィール情報を取得してた場合は、UserInfo API 叩くように変更しましょう。

ま、これだけっちゃこれだけなんですが、4/20までって言われると、焦りますね。

みんな realm が何かとか把握してるんでしょうか？

ってことで、自分が使ってるサービスが Google OpenID 2.0 使い続けてる場合は、サービス運営者に移行予定を問い合わせるなりしたほうが良いですね。

国内サービスの場合は、このブログのコメント欄にサービス名列挙していただけると、こちらでも対応可能かもしれません。(たぶん OpenID Foundation Japan 経由で)

あと、自社サービスが Google OpenID 2.0 使ってて移行ムリゲーとかいう事業者の方は、[OpenID Foundation Japan に問い合わせ](http://openid.or.jp/inquiry/) いただいけるといいかもですね。

要望が多ければ、OpenID Foundation Japan のエヴァンジェリスト達で Migration Handson なり Hackathon なり企画してもいいかなと思っています。

いや、何も[いますぐ Foundation に加盟](http://openid.or.jp/about/index.html#op-about-joining)しろなんていいませんよ？