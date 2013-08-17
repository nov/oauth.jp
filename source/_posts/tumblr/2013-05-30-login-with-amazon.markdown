---
layout: post
title: Login with Amazonを使うと、あなたのサイトのユーザーがアカウントハイジャックされる。
tags:
---
<p>Amazon が、<a href="http://login.amazon.com">Login with Amazon</a> というAPIおよびそれに付随する JS / iOS / Android SDKs を出してきました。</p>
<p>以下の記事にあるように、OAuth 2.0 ベースで認証連携を行う仕組みです。</p>
<p><a href="http://itpro.nikkeibp.co.jp/article/NEWS/20130530/480782/">Amazonアカウントでアプリやサイトにログイン、「Login with Amazon」を提供開始：ITpro</a></p>
<p>しかし、この Login with Amazon、まさに僕が以前以下の記事で紹介した「Facebook ID でログイン」の間違った実装方法を、公式ドキュメントで推奨してしまっています。</p>
<p><a href="http://www.atmarkit.co.jp/ait/articles/1209/10/news105_2.html">デジタル・アイデンティティ技術最新動向（2）：RFCとなった「OAuth 2.0」――その要点は？ (2/2) - ＠IT</a></p>
<p>この Amazon のドキュメントを読んで (or 「Facebook ID でログイン」を実装した経験を元に) Login with Amazon を自身の Web Site や iOS / Android アプリに導入すると、まず確実にあなたのサイト / アプリのアカウントがハイジャックされるような脆弱性を生むことになります。</p>
<p>@IT の記事からのリンクは OAuth.jp の Posterous -&gt; Tumblr 移行に伴いリンク切れになってしまっているので、ここにも OAuth 2.0 Implicit Flow を認証連携に使うことの危険性を説明するエントリーをリンクしておきます。</p>
<p><a href="/blog/archives/20120208-ios-sdk">"なんちゃら iOS SDK" でありそうな被害例</a></p>
<p>Facebook の場合は、上の記事にあるような response_type=code+token を iOS / Android アプリでは使えないので、以下の記事末尾にあるような対処法も用意されています。</p>
<p><a href="/blog/archives/20120207-oauth-20-implicit-flow">「OAuth 2.0 (Implicit Flow) でログイン」の被害例</a></p>
<p>が、ざっと Login with Amazon のドキュメントを読んだ限り、そのような対処法が一切用意されていない。</p>
<p>これはもう、Login with Amazon を導入した時点で、Amazon 側の対応無しではあなたのサイト / サービスの脆弱性を止めることができないということです。</p>
<p>既に一部有識者より Amazon 側に連絡が行っているようなので、さすがにこのまま Amazon が何の対応も取らないということは考えにくいですが、Amazon が何らかのアップデートを出してくるまで、Login with Amazon を使ってはいけません。</p>
<p>ご注意を。</p>
<p>[追記 2013.06.04]</p>
<p>Login with Amazon の問題点、解決されました。詳しくは以下の記事を。</p>
<p><a href="/blog/archives/20130604-login-with-amazon">Login with Amazon、もう使っても大丈夫！</a></p>
