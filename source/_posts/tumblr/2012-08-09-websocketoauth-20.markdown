---
layout: post
title: WebSocketでOAuth 2.0を使ってみた
tags:
- JustMigrate
---
<p>WebSocket Serverでユーザー認証とか、WebSocket Serverの裏にRailsなりのAPIがあってNode.jsからそのAPI叩く見たいなのはそれなりにありそうなのに、あんまサンプルコード的なものが見当たらなかったので、自分で書いてみたのをここに公開することにします。</p>
<p>ClientがWebSocket Serverにつなぐ時、<em><a href="http://socket.example.com/?access_token="><a href="http://socket.example.com/?access_token=">http://socket.example.com/?access_token=</a></a>***</em> みたいなURLにアクセスして、ServerはそのToken元にユーザーを認証して、その後socketがつながってる間はNode.jsがそのtoken使って裏のRails APIにアクセスする、という使い方をしてます。</p>
<p>Access Tokenの保存先にMySQL使ってるのはあくまでRails側でそっちのが手っ取り早かったからで、ここはRedisでもMongoDBでもなんでもいいです。この例ではNode.js側ではAccess Tokenに紐づいたClientとかScopeとかを検証してないし。</p>
<p><a href="https://gist.github.com/3300227"><a href="https://gist.github.com/3300227">https://gist.github.com/3300227</a></a></p>
<p>ところで、WebSocketではCookieベースの認証方式が主流なんですかね？</p>
