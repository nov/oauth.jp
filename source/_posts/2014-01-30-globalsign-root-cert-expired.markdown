---
layout: post
title: "Y!J API が止まった日 - GlobalSign の Root 証明書切れから学んだこと"
date: 2014-01-30 11:28
comments: true
categories:
---

昨日あたりから、Yahoo! Wallet や YConnect といった、Yahoo! Japan の API にアクセスできなくなったって人、ちらほらいるかもしれませんね。

僕もちょっとそういうケース見かけました。

なんか Yahoo! Japan がポカしちゃったの？とか、まぁ昨日まで健康に動いてたシステムが突然 Yahoo! Japan の API にアクセスできなくなっちゃったんだし、そらそう思うのもムリはない。

<strong>が、今回のケース、Yahoo! は全く悪くない！<br>プライバシーフリークはどうかと思うがな！！</strong>

では早速、今回起こったことを、振り返ってみましょう。

### Yahoo! API にアクセスできなくなった

Yahoo! Japan は、<b>yahoo.co.jp</b> 以外にも、CDN 用や API 用など、用途ごとにいくつかのドメインを持ってます。
今回止まったのは、その中の API 用の <b>*.yahooapis.jp</b> というドメイン。

Yahoo! Wallet はよく知らないけど、YConnect だと <b>userinfo.yahooapis.jp</b> っていうドメインがあって、そこにアクセスできなくなった。

ただし、API サーバーが止まったとかそういうのではなく、API にリクエスト投げる側での、SSL エラーによって。

<!-- more -->

### どんな SSL エラーが起こっていたのか？

<script src="https://gist.github.com/nov/8702799.js"></script>

うん...なんかエラーでてるね！w

<code>SSL certificate problem, verify that the CA cert is OK.</code> ですって！

でもぶっちゃけこれでどこが悪いかとか、何すればいいのかとか、サッパリですよね！

SSL まわりのエラー、たいていいつもそんな感じですよね。

そんなとき、来ました！Yahoo! Japan Tech Blog！

<a href="http://techblog.yahoo.co.jp/maintenance/4/">WebAPIやOpenIDでSSLエラーが起きる現象につきまして - Yahoo! JAPAN Tech Blog</a>

### GlobalSign の Root 証明書が、期限切れ！？

Yahoo! Japan Tech Blog 読んで問題理解できる人どれくらいいるのかはよくわかりませんが、まぁ結論としては、リクエスト元のマシンに bundle されてる GlobalSign の Root 証明書が古かったと。

で、今回某マシン上にあった期限切れ GlobalSign Root 証明書が、これ。

<script src="https://gist.github.com/nov/8703001.js"></script>

僕の手元のマシンにある、最新 (といっても発行されたのはもう数年も前) の GlobalSign Root 証明書はこちら。

<script src="https://gist.github.com/nov/8702954.js"></script>

違いは "<b>Not After : ...</b>" ってところ。

ちなみにどちらの Root 証明書も、そこに含まれてる公開鍵自体は同じなので、こいつの対になってる秘密鍵も同じ。
公開鍵が同じなのは、こんな感じで確認できる。

<script src="https://gist.github.com/nov/8699469.js"></script>

鍵ペア自体は同じなので、その秘密鍵で署名された証明書達 (GlobalSign の中間証明書達) はどちらの Root 証明書を使っても verify できるし、その中間証明書と紐づく秘密鍵で署名されている Yahoo! 等の事業者が持ってる証明書も、どちらの Root 証明書を使っても verify できる。

というか、できていた。古い方が expire するまでは。

もうちょっと細かい話をすると、例えば <b>userinfo.yahooapis.jp</b> の SSL 証明書を検証する場合だと、大雑把に言うと以下のようなフローになる。

1. <b>userinfo.yahooapis.jp</b> の証明書 (<b>*.yahooapis.jp</b> 向けマルチドメイン証明書) を取得
2. 1) の証明書が local にある信頼された証明書リストに含まれるかチェック => 通常は含まれてない
3. 1) の証明書を直接信頼することはできないので、1) の証明書の発行者を検証するため 1) の証明書から発行者情報取得
4. <b>*.yahooapis.jp</b> 証明書発行者の証明書を取得 (GlobalSign の中間証明書)
4. 4) の証明書が local にある信頼された証明書リストに含まれるかチェック => 通常は含まれてない
5. 4) の証明書を直接信頼することはできないので、4) の証明書の発行者を検証するため 4) の証明書から発行者情報取得
6. GlobalSign 中間証明書発行者の証明書を取得 (GlobalSign Root 証明書)
7. 6) の証明書が local にある信頼された証明書リストに含まれるかチェック => <b>これは含まれてる！！</b>

で、7) の証明書が valid であれば、1) の証明書も valid になる。

そして、7は valid だった。expire するまでは。

でも、expire しちゃったんですね、このタイミングで。<br>
<b>Not After : Jan 28 12:00:00 2014 GMT</b>

日本時間で、28日の夜9時。

この時を境に、古い方の GlobalSign Root 証明書を持ってるマシンでは、*.yahooapis.jp はじめ GlobalSign 発行の SSL 証明書を持ってるいろんなサーバーにアクセスする際に、SSL エラーが発生しだしたんですね。

...そらまぁ、Root 証明書 Expire してますし、エラーですよね...

### 何が起こっていて、何は起こっていなかったのか。

起こってたこと

* *.yahooapis.jp アクセス時の SSL エラー
* 一部の古い OS 積んでるようなマシンに bundle されてる GlobalSign Root 証明書の期限切れ

起こってなかったこと

* Yahoo! の API サーバーが止まった
* *.yahoo.co.jp アクセス時の SSL エラー
* Yahoo! Japan 側での SSL 証明書更新、設定変更
* その他なんらかの Yahoo! 側での異常
* OAuth 関連のエラー

ようするに、あれだ。

<b>お前らいつまで Debian5 (とか CentOS5 とか) 使ってんだよ</b> と。

まぁちゃんと Root 証明書リストとか必要に応じてアップデートできるんなら別だけど、ムリなんだったらやっぱ新しい OS 使おうぜ、と。

### 僕がやったこと

僕が手を下さないと行けなかったのは、たいしたサーバーじゃなかったし、さくっと <code>/etc/ssl/certs/GlobalSign_Root_CA.pem</code> を vi で書き換えるという荒技を使いましたよ。

まさか Root 証明書を vi で書き換える日が来るなんてね。

### 得られた教訓

ぶっちゃけ、Root 証明書って Expire すんだな、ってしみじみ思いましたよ。

落ち着いて考えると、そらまぁするんでしょうけども。

あと GlobalSign、もちょっと期限切れ直前におっきな声でアラートあげてもいんじゃないかなぁ？

今回の件は、厳密には Y!J も GlobalSign も悪くないし、ある意味天災的なもんかとも思いますが、でもこの件で GlobalSign から証明書買おうとする人は、一時的には減るよね。

だって古い OS からはそのままだと SSL エラーでるようになっちゃったんだもの、GlobalSign 発行の証明書。

他にも証明書発行業者が複数あるなかで、それは結構なビジネスリスクですよね。

まぁどんなキャンペーンやればみんなが Debian5 -> Debian6 に上げるのか、アイデア無いですけども...

<br>
<br>
<br>

そして、もう一つ、僕らは重要なことを学んだ気がする。

<strong style="font-size: 2.5em">PKI、年取ったな。</strong>

<br>
<br>
<br>

ps1.

ちなみに、今後数年はメジャーな Root 証明書が Expire することなさげです。気づいたらその数年あっという間に過ぎてんでしょうけど。

<br>
<br>
<br>

ps2.

タイトルの最初で言っといてなんですが、Yahoo! Japan の API は止まってません。
Tech Blog といい、Yahoo! 側では何の問題も起こしてないのに、いろいろこまめに対応しててすばらしいなと思います！

<strong>プライバシーフリークはどうかと思うけどな！</strong>