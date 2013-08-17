---
layout: post
title: WebIntent x OpenID Connect
tags:
---
<iframe src="http://www.slideshare.net/slideshow/embed_code/12296336" width="400" height="334" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe><br/><p>WebIntentsを使ってOpenID ConnectのDiscoveryをやってみました。</p>

<p>試すには、以下のステップを踏んでください。</p>

<ol><li><a href="https://connect-op.heroku.com/">https://connect-op.heroku.com/</a> にアクセス (ここにintentタグが仕込まれてます)</li>
<li><a href="https://connect-rp.heroku.com/">https://connect-rp.heroku.com/</a> にアクセス</li>
<li>"Or Try WebIntents?" というボタンをクリック</li>
</ol><p>するとPopupが開いて、Step1で登録されたNov OPが選択肢に現れるので、それを選択するとNov RPに戻ってDiscovery結果がAlertで表示され、その後通常のOpenID Connectのログインフローに進みます。</p>

<p>ps.
webintents.orgのJSの問題なのか、なぜかSafariでしか動きません。</p>
