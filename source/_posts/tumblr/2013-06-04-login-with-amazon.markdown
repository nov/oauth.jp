---
layout: post
title: Login with Amazon、もう使っても大丈夫！
tags:
---
<p>先日以下のような記事を書きました。</p>
<p><a href="/blog/2013/05/30/login-with-amazon">Login with Amazonを使うと、あなたのサイトのユーザーがアカウントハイジャックされる。</a></p>
<p>Login with Amazon 危険ですよ！という記事だったのですが、その後 Login with Amazon のドキュメントに、Token Info API が追加されました。</p>
<p>こちらの Step4 Obtain Profile Information のサンプルコードで、Profile API を叩く前に Token 発行先の Client ID が自身の Client ID と一致することを確認する処理が追加されています。</p>
<p><a href="http://login.amazon.com/website">Web - Login with Amazon Developer Center</a></p>
<p>こちらのより詳細なドキュメントには、Security Considerations の &#8220;Impersonating a Resource Owner in Implicit Flow&#8221; という節で、Token Info API を使わない場合の攻撃例が述べられています。</p>
<p><a href="https://images-na.ssl-images-amazon.com/images/G/01/lwa/dev/docs/website-developer-guide._TTH_.pdf">Login with Amazon - Developer Guide for Websites (PDF)</a></p>
<p>というわけで、Login with Amazon、ドキュメント通り実装すれば安全です :)</p>
