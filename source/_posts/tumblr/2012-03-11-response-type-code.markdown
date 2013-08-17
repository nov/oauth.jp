---
layout: post
title: OAuth 2.0でユーザーが認可をする”アプリケーション”とはサービス全体のことではない - r-weblife
tags:
---
<blockquote><p>response_type=code tokenとかあるので仕様曖昧だよな～と思っていたのですが、普通に書いてありました。</p>

<p>A client application consisting of multiple components, each with its
own client type (e.g. a distributed client with both a confidential
server-based component and a public browser-based component), MUST
register each component separately as a different client to ensure
proper handling by the authorization server.</p>

<p>Clientはユーザーに代わって保護リソースにアクセスするアプリケーション。
Client Typesとして、秘密鍵を安全に管理できるconfidential、クライアントサイドで動いて管理できないのはpublic。
ConfidentialならAuthZ Code, PublicなのはImplicit使え。
サーバーサイドとクライアントサイドの両方のComponentから構成されている場合、別々で登録しろ！</p></blockquote>&#8212;<p><a href="http://d.hatena.ne.jp/ritou/20120311/1331444771">OAuth 2.0でユーザーが認可をする”アプリケーション”とはサービス全体のことではない - r-weblife</a></p>

<p>で、そういう状況でどうやって効率的に同意を取るかは、AuthZ Serverの実装依存、か。。。</p>

<p>(via <a href="http://kyomichi.tumblr.com/" class="tumblr_blog">kyomichi</a>)</p>
