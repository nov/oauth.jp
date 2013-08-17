---
layout: post
title: Rack::OAuth2 で token_type=mac をサポート
tags:
- JustMigrate
---
<p>@nov です。</p>
<p>先日 @ritou が「<a href="http://d.hatena.ne.jp/ritou/20110418/1303055521">OAuth 2.0 MAC Tokenを使った署名付きリクエストの作り方</a>」という記事を書いてたので、それを参考に Rack::OAuth2 でも MAC サポートを追加しました。</p>
<p>Sample Server はこちらです。</p>
<p><a href="http://rack-oauth2-sample-mac.heroku.com/"><a href="http://rack-oauth2-sample-mac.heroku.com/">http://rack-oauth2-sample-mac.heroku.com/</a></a></p>
<p><a href="https://github.com/nov/rack-oauth2"><a href="https://github.com/nov/rack-oauth2">https://github.com/nov/rack-oauth2</a></a></p>
<p>そして早速 @ritou が Sample Client を作ってくれました。（おかげで Bug が一つ見つかりました。Thanks!）</p>
<p><a href="http://www8322u.sakura.ne.jp/oauth2sample/"><a href="http://www8322u.sakura.ne.jp/oauth2sample/">http://www8322u.sakura.ne.jp/oauth2sample/</a></a></p>
<p>ついでに Rack::OAuth2 にも AccessToken モデルを用意しました。</p>
<p>Bearer、MAC ともに使い方はこんな感じです。</p>
<p><a href="https://gist.github.com/933962"><a href="https://gist.github.com/933962">https://gist.github.com/933962</a></a></p>
<p>そして密かに Legacy なアクセストークン (Facebook および mixi を想定) もサポートしだしました。 Bearer とか MAC とか言っても、誰もそんなの使ってないですからね。。</p>
<p>Legacy の方も、こんな感じで使えるんでは無いでしょうか？（適当w）</p>
<p><a href="https://gist.github.com/933954"><a href="https://gist.github.com/933954">https://gist.github.com/933954</a></a></p>
<p>これで OAuth2 gem から Rack::OAuth2 に、いつでも乗り換えられますね ;)</p>
