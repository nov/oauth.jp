---
layout: post
title: Facebookが新たなAccess Token Introspection APIを出したようです
tags:
- JustMigrate
---
<p>今朝fb_graphにこんな要望が来てて知ったのですが、Facebookが新たなAccess Token Introspection APIを出したようです。</p>
<p><a href="https://github.com/nov/fb_graph/issues/269">Adding support for /debug_token endpoint · Issue #269 · nov/fb_graph</a></p>
<p>こんなリクエストを送ると</p>
<p><a href="https://gist.github.com/3896829"><a href="https://gist.github.com/3896829">https://gist.github.com/3896829</a></a></p>
<p>こんなレスポンスが帰ってくる、と。</p>
<p><a href="https://gist.github.com/3896826"><a href="https://gist.github.com/3896826">https://gist.github.com/3896826</a></a></p>
<p>"data" ってなんやろとかApp Token無いと使えへんのメンドイなとか思ったりはしますが、ちゃんと別のClientに発行されたAccess Tokenを<code>input_token</code>として送るとエラーが帰ってくるし、Success Responseにはscopeとかissued_atとかも付いてくるので、いろいろと使い勝手は良さそうですね。まぁ例のごとく完全に独自仕様ですが。</p>
<p>って実際にGraph API Explorerで試したら、issued_at返ってこないですが（謎</p>
<p>オフィシャルドキュメントはこちら。</p>
<p><a href="https://developers.facebook.com/docs/howtos/login/debugging-access-tokens/">Debugging Access Tokens and Handling Errors</a></p>
