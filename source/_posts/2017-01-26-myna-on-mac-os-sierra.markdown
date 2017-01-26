---
layout: post
title: "MacOS Sierra (10.12.x) でマイナポータルにログイン"
date: 2017-01-26 21:37
comments: true
categories:
---

お久しぶりです、nov です。

おかげで無事12月末で [YAuth.jp](http://yauth.jp) も初年度を終え、2期目に突入しております。

MVP は意気込み送ってこいや的なメールが来ており、いいかげんそろそろ Windows 10 でドメインジョイン (正直 UX とかよくわかってない) せんとなぁとか思いつつ、今日は MacOS です。

MacOS でマイナポータルにログインするお話です。

みなさん、知ってましたか？マイナポータルって OpenID Connect の IdP なんですよ！もしかしたら Access Token とか払い出しちゃう機能なんかもあるんですよ！

ということで、まぁその辺を調べるにも、ログインせんことには何も始まりません。

**え？マイナンバーカード持ってないだって？？んなやつぁしらん！！！**

そんな人は [NIST SP 800-63-3 翻訳版](https://openid-foundation-japan.github.io/800-63-3/index.ja.html) でも読んで、Identity Proofing とかハードウェアトークンとかに思いをはせつつ市役所いって取って来てください。

マイナンバーカード取る時、NIST SP 800-63-3 の3ページ目テストに出ますからね。

<!-- more -->

*嘘です。*

## MacOS Sierra でマイナポータルにログイン

さて、Windows 10 でログイン、できるに決まってますよね。

e-Tax にログインできるんなら、おなじような仕組みのマイナポータルにだってログインできるに決まってます。

ということで、今日は e-Tax から見放された MacOS Sierra でマイナポータルにログインしてみましょう。

クソ長い PDF を真面目に読まれる方は、[公式マイナポータルログイン手順](https://myna.go.jp/SCK0101_03_001/SCK0101_03_001_Init.form) をご覧ください。

### カードリーダー買う & ドライバーインストール

マイナンバーカードリーダーのアフィリンクを貼りたいがためにこの記事を書いているといっても過言ではありません。

このサンワサプライのカードリーダーは、[MacOS Sierra 対応のドライバ](https://www.sanwa.co.jp/support/download/dl_driver_ichiran.asp?code=ADR-MNICUBK) あります。

[![サンワサプライ 接触型ICカードリーダライタ ADR-MNICUBK](https://ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=B01M2DJ9WG&Format=_SL160_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=bianca0b-22)](http://amzn.to/2j7WSEx)

マイナポータル公式の対応カードリーダー表には El Capitan の欄はあるけど Sierra の欄がなく、そこみてもどれが Sierra 対応なのかよくわかりません。

NTT コミュニケーションズのやつはドライバが El Capitan までだったりしたので、Sierra 対応カードリーダー選ぶのが最初のハードルです。

さて、Amazon さんから [サンワサプライ 接触型ICカードリーダライタ ADR-MNICUBK](http://amzn.to/2j7WSEx) 届いたら、Sierra 対応のドライバインストールしてください。

この記事執筆時点では、以下の一番下のやつです。

[![ADR-MNICUBK Deivers](/images/posts/myna/card-reader-drivers.png)](https://www.sanwa.co.jp/support/download/dl_driver_ichiran.asp?code=ADR-MNICUBK)

これでカードリーダーのセットアップは完了です。

### Java 8 Update 121 にアップデート

[マイナポータル動作環境について](https://img.myna.go.jp/html/dousakankyou.html) によると、Java Version 8 Update 111 以上が必要なようです。

あと、Java のアップデートするたびに、ポリシーファイルとかいうのが書き換えられて、再度なんか設定し直しらしいです。

ということで、とりあえず現時点での最新の Java 入れときましょう。

![Java 8 Update 121](/images/posts/myna/java8-121.png)

Oracle Java 入れたことあれば　MacOS の設定アプリに屈辱の "Java" メニュー現れてるはずなんで、そっからアップデートできるはずです。

入れたこと無い人は、"MacOS Java 8" とかでググるなりして入れてください。

屈辱です。

### JPKI 利用者クライアントソフトのインストール & 初期設定

続いて [マイナポータル動作環境について](https://img.myna.go.jp/html/dousakankyou.html) の「JPKI利用者クライアントソフトの準備」からもリンクされてる [JPKI利用者クライアントソフト](https://www.jpki.go.jp/download/index.html) の MacOS 版をインストールします。

こいつはちょっと時代の流れに付いてくるのがゆっくりなので、「JRE 8.0 Update111」を推奨して来ますが、Java 8 Update 121 では少なくとも動きます。

あ、あと「Smart Card Services」は入れちゃダメです！

いや、不要なだけで入れてもいいのかもです。

まぁとにかくマイナンバーカード使う場合は「Smart Card Services」はいりません。

ということで、入れるのは　[JPKI利用者クライアントソフト - Macintosh をご利用の方](https://www.jpki.go.jp/download/mac.html) の一番下の方の「利用者クライアントソフトのダウンロード」ってとこにリンクされてるやつです。

*いや、ダウンロードリンク下すぎでしょ。*

で、インストール終わるとこんなの出て来ます。

よくわかんないけど「はい」にしときましょう。

![JPKI 証明書有効期限通知設定](/images/posts/myna/jpki-cert-expiry-notification.png)

Retina 対応してないやつ、久々にみました。

最後にインストールされた全アプリが表示されます。

![JPKI アプリ一覧](/images/posts/myna/jpki-applications.png)

この中の「Java実行環境への登録」ってのだけ実行しといてください。

### マイナポータル環境設定プログラムのインストール & 設定

さて、JPKI 利用者クライアントソフトの設定が終わったら、また [マイナポータル動作環境について](https://img.myna.go.jp/html/dousakankyou.html) に戻って、今度は「マイナポータル環境設定プログラムの実施」ってとこから「[環境設定プログラム (Mac 用)](https://img.myna.go.jp/tools/mac/MyNASetup.pkg)」をインストールします。

こいつはインストール後に特に何かを主張したりはしません。

### ブラウザの制限をちょっと弱める

最後に [マイナポータル環境設定プログラム - ご利用方法マニュアル](https://img.myna.go.jp/manual/2.pdf) の P.20〜P.25 だけちゃんと読みながら、ブラウザのセキュリティ制限を少しずつ弱めます。

P.24 の「安全でないモード」とか、なんていう裏技感。

あ、ちなみに Chrome はサポートされてません。

Safari 使ってください。

### いよいよマイナポータルログイン

さて、ここまで設定できたら、Safari を再起動して、マイナポータルトップ画面のログインボタンをクリックしてください。

マイナンバーカード取得時に設定した PIN コードの入力を求める Popup が出て来たら、成功です。

![マイナポータルログイン](/images/posts/myna/myna-pin-input.png)

PIN を入れてログイン成功したことを確認したら、あとはご自由にマイナポータルで遊んでみてください。

特にまだ何ができるわけでもないですけどね。

### 最後に

ここまで来たあなたは、ブラウザのセキュリティ制約をいくつか解除してしまっています。

マイナポータルに満足したら、ちゃんと元どおり 3rd-party Cookie は「閲覧した Web サイトは許可」、Popup はブロック、Java プラグインは無効化しておきましょう。

いや、もっと厳しくしてもいいし、そのまんまでいいならそれも自己判断でいいんですけどね。

では、素敵なマイナライフを！！
