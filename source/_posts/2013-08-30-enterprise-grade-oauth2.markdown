---
layout: post
title: "エンプラグレード OAuth 2.0"
date: 2013-08-30 00:27
comments: true
categories:
---

エンプラ、よく分かりません。

エンプラグレード、逆に不安です。

と、まぁそんな内輪ネタはさておき、最近はエンプラで OAuth 2.0 使う事例も増えてるようです。

Google Apps とか導入してたら、OAuth 2.0 とか G+ Sign-in (= OpenID Connect) も使うでしょうし、そんな時にエンプラだと特定のアプリにだけ特定の API へのアクセスを許可したい、なんてこともあるんでしょう。

で、今日それっぽい事例をちょっと見かけたので、response_type=code ってなってるとこを response_type=token に書き換えてやったんですが...

しっかり access_token 取れて、自分で書いたスクリプトからも API アクセスできましたよ！

特定アプリに限定したつもりでも、別に自分のアカウントと紐づいた access_token 取るだけなら client_secret いらなかったりするのは、当然っちゃ当然なのですが、これがエンプラの世界では許容されるのかされないのかがよく分かんないです。

もし許容されないのだとしたら、特定のアプリに限定するってなら response_typ=code に限定するとかもしないとダメなんでしょうねぇ。

と、Octopress に移行して断然ブログ書きやすくなったので、つらつら書いてみました。

== 以下ステマ ==

ん？「エンプラで OAuth 2.0」ってのに興味湧きました？

そんなあなたに [OpenID TechNight vol.10](http://openid.doorkeeper.jp/events/5373)！

今回の TechNight は日本最先端のエンプラ ID 厨たちが集うエンプラ特集。

先日アメリカで開催された世界のエンプラ ID 厨が集う [Cloud Identity Summit (CIS) 2013](http://www.cloudidentitysummit.com) に参加した日本のエンプラ ID 厨たちが、CIS で感じた世界のエンプラ ID ビジネス／テクノロジーの動向を熱く語ります。たぶん。

てかなんだここ最近のエンプラ ID 厨の盛り上がりは！

エンプラ ID 厨向けの TechNight に既に申し込み100名超えてるとか (((((( ;ﾟДﾟ)))))ｶﾞｸｶﾞｸﾌﾞﾙﾌﾞﾙ

ps.

関西在住でさすがにそんな気軽に関東に来れないって方には、[ID & IT 2013](http://nosurrender.jp/idit2013/) の大阪会場に参加するという手も。