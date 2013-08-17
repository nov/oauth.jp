---
layout: post
title: rack-oauth2というOAuth 2.0のServer側のライブラリをリリースしました
tags:
- JustMigrate
---
<p>そろそろ毎回「@nov です」で始めるのが寂しくなってきましたが、@novです。</p>
<p>表題の通りです。</p>
<p>ソースはこちら。</p>
<p><a href="http://github.com/nov/rack-oauth2"><a href="http://github.com/nov/rack-oauth2">http://github.com/nov/rack-oauth2</a></a></p>
<p>こんな感じで使ってやってください。</p>
<p><span style="font-family: helvetica, arial, freesans, clean, sans-serif; line-height: 18px;"> </span></p>
<h4 style="line-height: 1.4em; padding: 0px; margin: 0px;">Resource Owner Authorization &amp; Token Endpoint</h4>
<p><a href="https://gist.github.com/584594"><a href="https://gist.github.com/584594">https://gist.github.com/584594</a></a></p>
<p><span style="font-family: helvetica, arial, freesans, clean, sans-serif; line-height: 18px;"> </span></p>
<h4 class=" aptureTMMSelection" style="line-height: 1.4em; padding: 0px; margin: 0px;">Protected Resource Middleware Setting</h4>
<p><a href="https://gist.github.com/584565"><a href="https://gist.github.com/584565">https://gist.github.com/584565</a></a></p>
<p>基本的には、「エラーハンドリングとかレスポンスの整形とかはrack-oauth2がやるんで、ポリシーの部分とかを自分たちで決めて使ってください。」というスタンスです。</p>
