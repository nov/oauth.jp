---
layout: post
title: OAuth2 gemを使ってmixi Graph APIにアクセス
tags:
- JustMigrate
---
<h2>OAuth2 gemのインストール</h2>
<p>現状OAuth2 gemはOAuth 2.0の古いspec (draft 0?) に準拠しているので、mixiがサポートしているdraft 10に対応したOAuth2 gemをここからgit clone &amp; rake installする。</p>
<p><a href="https://github.com/nov/oauth2"><a href="https://github.com/nov/oauth2">https://github.com/nov/oauth2</a></a></p>
<p>あとはこんな感じ。</p>
<p><a href="https://gist.github.com/803773"><a href="https://gist.github.com/803773">https://gist.github.com/803773</a></a></p>
<h3>注意事項</h3>
<p>mixiのドキュメントの「<a href="http://developer.mixi.co.jp/connect/mixi_graph_api/api_auth#toc-api1">認証認可手順 &gt; リフレッシュトークン、アクセストークンの入手</a>」によると、mixiのaccess tokenは15分でexpireされ、refresh tokenも「常に同意する」のチェック有無によって3ヶ月 or 6時間でexpireされるので、TwitterのOAuthの用にaccess tokenをずっと使い続けることはできません。</p>
<p>あと、&#8221;w_voice&#8221; のパーミッションを取得しても、&#8221;r_voice&#8221; のパーミッションは付いてこないので注意。</p>
