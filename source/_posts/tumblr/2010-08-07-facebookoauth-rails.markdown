---
layout: post
title: FacebookのOAuthを使うサンプルアプリ (Rails)
tags:
- JustMigrate
---
<p><a href="http://twitter.com/nov">@nov</a> です。</p>
<p>自分で作ってる <a href="http://github.com/nov/fb_graph">fb_graph</a> という Facebook Graph API の Ruby ライブラリを OAuth 対応したついでに、Railsでサンプルアプリを作りました。</p>
<p><a href="http://fbgraphsample.heroku.com/">デモサイトはこちら。</a></p>
<p><a href="http://github.com/nov/fb_graph_sample">ソースコードはこちら。</a></p>
<p>ソースコード内には localhost:3000 で使える API Key&amp;Secret も埋め込んであるので、特に Facebook にアプリ登録しなくても動きます。（localhost:3000 以外で動かす場合は自分でアプリ登録してください）</p>
<p>いまのところRailsのバージョンは3.0.0.RCです。</p>
<p>app/models/facebook.rb と app/controllers/facebooks_controller.rb を見れば、Facebook 独自の JavaScript SDK と通常の OAuth2 の Web Server provile の使い方が分かるかと思います。</p>
