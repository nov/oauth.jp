---
layout: post
title: "Facebook Login で Covert Redirect を防止する"
date: 2014-05-08 14:16
comments: true
categories:
---

[OAuth 2.0 Implicit Flow では Covert Redirect 経由で access token が漏れる件](/blog/2014/05/07/covert-redirect-in-implicit-flow/)については既に紹介しましたが、ここではみなさん大好き Facebook Login で OAuth Client Developer ができる対策について紹介することにします。

### 攻撃方法

1. 攻撃対象となる OAuth Client の FB client_id を取得
    * client_id は FB Login ボタンさえクリックすればアドレスバーに表示される。
2. Authorization Request URL を構築
    * 対象 Client が通常 response_type=code を使ってる場合でも、問答無用で response_type=token にしてください。
    * [https://www.facebook.com/dialog/oauth?client_id=*<client_id>*&redirect_uri=*<redirect-url>*&response_type=token](https://www.facebook.com/dialog/oauth?client_id=relace-me&redirect_uri=relace-me&response_type=token)
3. 被害者を2で作った URL にアクセスさせます
    * フィッシングがんばってください。
4. redirect_uri に指定しておいた URL で fragment に付いて来る access token を待ち構えます。

以上、巧いフィッシング方法さえ思いつけば簡単ですね。

### 防御方法

防御方法は、大きく分けて3つあります。

1. open redirector を撤廃する
2. response_type=token を利用禁止にする
3. redirect_uri を完全一致しか認めさせなくする

<!-- more -->

#### open redirector を撤廃する

まぁそらそれがベストですよね。

「あっ、はい」とか言う人いるけども。

まぁ、「それができれば最初っからやってるっちゅうねん」ってのは、ごもっとも。

#### response_type=token を利用禁止にする

FB にはそのためのオプション "[appsecret_proof](https://developers.facebook.com/docs/graph-api/securing-requests/)" があります。

厳密には response_type=token を利用禁止にするのではなく、access token 単体で API Request をさせない、というオプションです。

このオプションを有効化すると、API Request に client_secret が必要になるため、client_secret が漏洩しない限りは、たとえ access token が漏洩してしまっても問題ありません。

え？client_secret が漏洩するかもって？

そんな、OAuth 1.0 じゃあるまいし、まさか Native App に client_secret 埋め込んだりしてないはずですよね？ね？？

なおこのオプションを有効にできるのは、client_secret を秘匿に保てる Client、つまりサーバーサイドで動作する Web App に限られます。

Client が Native App な場合や、Web App と Native App の両方で同じ client_id を使っている場合は、このオプションは選べません。

2014年5月現在では、このオプションは以下の URL から "App Secret Proof for Server API calls" というのを ON にすれば有効化されます。
https://developers.facebook.com/apps/:app_id/settings/advanced/

#### redirect_uri を完全一致しか認めさせなくする

デフォルトでは redirect_uri の部分一致を許容する Facebook ですが、redirect_uri を完全一致のみ許可させるよう指定することもできます。

2014年5月現在では、このオプションは以下の URL から "Valid OAuth redirect URIs" というのを field に URL を指定すれば有効化されます。
https://developers.facebook.com/apps/:app_id/settings/advanced/

Native App でも利用する client_id に関しては、このオプションを利用しましょう。

Web App でも、OAuth 2.0 の独自拡張である [appsecret_proof](https://developers.facebook.com/docs/graph-api/securing-requests/) を利用するよりも、OAuth 2.0 の仕様に乗っ取った挙動になるこのオプションの方が、ライブラリをそのまま使えるとかいろいろメリット多そうですね。

みなさんのユースケースにあったオプションをお選びください。