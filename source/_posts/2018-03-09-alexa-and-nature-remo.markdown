---
layout: post
title: "Alexa と Nature Remo の Account Linking"
date: 2018-03-09 20:24
comments: true
categories:
---

ついに [Amazon Echo Plus](http://amzn.to/2HjU7Yp) の購入券当選通知が来たので、Echo Plus と一緒に [Nature Remo](http://amzn.to/2oVl4Ls) を買いました。

<a href="https://www.amazon.co.jp/Nature-Inc-Remo-01-Remo/dp/B06XCQFP96/ref=as_li_ss_il?s=aps&ie=UTF8&qid=1520594772&sr=1-1-catcorr&keywords=nature+remo&linkCode=li3&tag=bianca0b-22&linkId=13d9bc61ca409c43ab2dc391c559aa12" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=B06XCQFP96&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=bianca0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=bianca0b-22&l=li3&o=9&a=B06XCQFP96" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

エアコン推しですが、テレビのリモコン等、赤外線飛ばすリモコンならなんでも対応できる、素敵な「スマートリモコン」です。

Nature Remo アプリでセットアップして、アプリからリモコンつけたり消したりできるのも素敵ですし、アプリで設定しといたら iPhone が Nature Remo から 30m 以上離れたら自動でエアコン切るルールとか設定しとけるのも素敵です。

が、やはりここは

**「Alexa、エアコンをつけて」**

ってやりたいですよね。

<!-- more -->

そのためには、Alexa アプリで「Nature Remo Smart Home Skill」ってのをインストールして、アカウントリンキングを設定する必要があります。Nature Remo に紐づいた Nature Remo アカウントと、Alexa に紐づいた Amazon アカウントを紐づける処理ですね。

で、これ、Alexa アプリから OAuth Authorization Request をスタートさせて、Nature Remo 側の Authorization Server にログインして、Alexa アプリ (というか Alexa 側の Redirect URI) に戻ってくる必要があります。

ここで、Nature Remo 側で必ずエラーになります。

Nature Remo アカウントって、パスワードがなくて、毎回登録したメアドに送られてくるマジックリンクをクリックしてログインするんですよ。

でね、Alexa アプリの SafariViewController でメアド入力して、外部 Safari でメール中のリンククリックすると、両者のセッションがずれちゃうから、CSRF エラーかなんかになってるんですよ。

US の Alexa の場合は、Amazon.com サイトから Skill インストールできるから、PC/Mac で設定してやればうまくいくのかもしれませんが、日本の場合は Alexa アプリからアカウントリンクスタートするしかないんで、これ絶対にログイン成功しませんよね...

でもまぁ、僕はちょっと OAuth とか iOS とか詳しいんで、やってやりましたよ。

絶妙なタイミングで、SafariViewController の下の方にある外部 Safari 開くボタンを、押してやりましたよ。

そう、OAuth Authorization Request が Remo サーバーに届く前に、外部 Safari に遷移してしまうんです。

で、なんとかうまくいきました。

**これ一般ユーザーには無理ゲーすぎやろ。**