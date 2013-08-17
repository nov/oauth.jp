---
layout: post
title: Rack::OAuth2 Sample Server
tags:
- JustMigrate
---
<p>@novです。</p>
<p>OAuth 2.0 draft v.13準拠のRack::OAuth2 version 0.3.0のリリースにあわせて、Rails3でSample Appを作りました。</p>
<p><a href="http://rack-oauth2-sample.heroku.com"><a href="http://rack-oauth2-sample.heroku.com">http://rack-oauth2-sample.heroku.com</a></a></p>
<p>Facebookアカウントでログインしたら、なんとなくで使い方分かると思います。</p>
<p>Access Token or Authorization Codeを取得した後は、こちらのClient Sampleみたいなのでアクセスしてください。</p>
<p><a href="https://gist.github.com/857277"><a href="https://gist.github.com/857277">https://gist.github.com/857277</a></a></p>
<p>このアプリはOAuth2 Serverを作る人向けなので、ソースの読み方も。</p>
<p>Authorization Server =&gt; app/controllers/authorizations_controller.rb</p>
<p>Token Endpoint =&gt; config/routes.rb (Rack App)</p>
<p>Resource Server =&gt; config/application.rb (Rack App) &amp; lib/authentication.rb</p>
