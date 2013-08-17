---
layout: post
title: 「OAuth 2.0 (Implicit Flow) でログイン」の被害例
tags:
- JustMigrate
---
<h3>。登場人物</h3>
<ul><li>OAuth 2.0対応してる某ゲームプラットフォーム</li>
<li>某ゲームプラットフォーム上で占いゲームを運営してる攻撃者</li>
<li>某ゲームプラットフォーム上で運営されてる農園ゲーム（= 被害アプリ）</li>
<li>某ゲームプラットフォーム上で無邪気に遊んでる被害ユーザ</li>
</ul><p>※ 念のため、今回の話は特にゲームに限った話ではない。</p>
<h3>前提</h3>
<p>某ゲームプラットフォーム、農園ゲーム共に、XSS とか CSRF とかセッションハイジャックされるような脆弱性はない。</p>
<p>農園ゲームはプラットフォームが発行するAccess TokenをOAuth 2.0のImplicit Flowを使って受け取り、同じくプラットフォームが提供するProfile API (GET /me とか) にアクセスして、レスポンスに含まれる user_id をもとにユーザを認証している。</p>
<p>攻撃者は占いゲームのDBから任意のAccess Tokenを取得可能。</p>
<h3>シナリオ</h3>
<ol><li>被害ユーザが占いゲームに登録 (= 占いゲームに対するAccess Tokenが発行される)</li>
<li>攻撃者が農園ゲームにアクセス、Authorization Flow開始</li>
<li>攻撃者が某ゲームプラットフォーム -&gt; 農園ゲームへのRedirect Responseを中断させる</li>
<li>攻撃者がFragment中のAccess Tokenを占いゲームのDBから持ってきた被害者のAccess Tokenに置き換え</li>
<li>農園ゲームは受け取ったAccess Tokenをもとに、攻撃者を被害ユーザとして認証</li>
<li>攻撃者が被害ユーザとして農園ゲームで遊ぶ</li>
</ol><h3>対策方法</h3>
<p>農園ゲームの開発者は、受け取ったAccess Tokenが農園ゲームに対して発行されたものであることを確認すること。占いゲームに対して発行されたAccess Tokenを受け入れないこと。</p>
<p>プラットフォームがFacebookの場合なら、以下のリクエストを行うことで、Access Tokenがどのクライアントに発行されたものかを検証可能。</p>
<div class="CodeRay">
  <div class="code"><pre>GET /app
Authorization: OAuth replace_with_your_access_token</pre></div>
</div>

<p>急募&#160;: 他のプラットフォームの場合の対応策</p>
<h3>参考文献</h3>
<ul><li><a href="http://www.sakimura.org/2012/02/1487/">単なる OAuth 2.0 を認証に使うと、車が通れるほどのどでかいセキュリティー・ホールができる</a></li>
<li><a href="http://d.hatena.ne.jp/ritou/20120206/1328484575">OAuth 2.0 Implicit Flowをユーザー認証に利用する際のリスクと対策方法について #idcon</a></li>
</ul>
