---
layout: post
title: サイボウズ Live APIがOAuth 1.0に対応したようです
tags:
- JustMigrate
---
<p>今朝Twitterで <a href="http://twitter.com/nakajiman/status/24887989061">@nakajiman</a> さんに教えて頂いたのですが、サイボウズ Live APIがOAuth 1.0を採用したようです。</p>
<p>サイボウズ Live APIのドキュメントはこちらです。</p>
<p><a href="https://developer.cybozulive.com/doc/current/pub/authorize.html"><a href="https://developer.cybozulive.com/doc/current/pub/authorize.html">https://developer.cybozulive.com/doc/current/pub/authorize.html</a></a></p>
<p>ドキュメントの中に出てくる用語がRFC版のものとコミュニティ版のものとまじっていて混乱するかもしれませんが、ruby-oauth使うとこんな感じになります。</p>
<p><strong>リクエストトークン (テンポラリクレデンシャル) 取得 &amp; ユーザ (リソースオーナー) をサイボウズLiveへリダイレクト</strong></p>
<p><a href="https://gist.github.com/586267"><a href="https://gist.github.com/586267">https://gist.github.com/586267</a></a></p>
<p><strong>アクセストークン (トークンクレデンシャル) 取得</strong></p>
<p><a href="https://gist.github.com/586272"><a href="https://gist.github.com/586272">https://gist.github.com/586272</a></a></p>
<p>xAuthもサポートしているようですが、ホワイトリストに関する記述はないですね。どのclientでも使える、のかな？あと、エラーを503で返されると、ライブラリは対応しきれないかも知れませんね。</p>
<p>ちなみにここにあるアクセスレベルってのは、client単位でscopeが設定される (=Y!方式) ってことでしょうかね？</p>
<p><a href="https://developer.cybozulive.com/doc/current/pub/overview.html"><a href="https://developer.cybozulive.com/doc/current/pub/overview.html">https://developer.cybozulive.com/doc/current/pub/overview.html</a></a></p>
