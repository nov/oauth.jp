---
layout: post
title: "OpenID Connectはそんなに大変かね？"
date: 2016-02-24 11:14
comments: true
categories:
---

[OAuth 2.0 + OpenID Connect のフルスクラッチ実装者が知見を語る - Qiita](http://qiita.com/TakahikoKawasaki/items/f2a0d25a4f05790b3baa) ってのになんかフォローアップしろよ的なのが来たので。

ざっと読んだ感想としては、「OpenID Connect の OPTIONAL な機能全部実装したら、そら大変ですね」という感じ。（Authlete に関しては、OpenAM みたいな感じで使われる、OpenAM よりはるかに簡単に使える代わりに有料の何かなんだろうな、というイメージです）

## OAuth は必要なのか？

* Basic 認証は死んだ。
* ユーザー単位での API のアクセスコントロールがしたいです。

っていう前提で話すると、OAuth 以外まともな選択肢が無いんじゃないでしょうか。

OAuth の各種 Extension (RFC 6749 & 6750 以外にいろいろある) に関しては、適宜必要なのを実装すればいいんだけど、どれが必要なのかを選ぶのが大変なのは事実で、そこのベストプラクティスとかユースケースごとのガイドラインは今後の課題。IETF OAuth WG の中の人たちも、それは認識している。

## 「OAuth 認証」とは

OAuth は「End-User (に信頼された OAuth Server) が OAuth Client のアクセス権限をコントロールする」というコンテキストにおいての標準化されたプロトコルであって、「Identity Provider (IdP) が End-User を認証した結果を受け取って、Relying Party (RP) が (IdP への信頼を元に) End-User を認証する」というコンテキストで OAuth を使うユースケースが「OAuth 認証」と呼ばれるやつです。

後者のコンテキストで、OAuth は何も標準的な仕様を定めてはいません。

IdP が End-User を認証した結果を RP に伝える方法 (ID Token 相当) や、そのコンテキストで求められることが多い認証されたユーザー属性情報の取得方法 (UserInfo 相当) については、完全に各 Platform が独自に API を提供してるだけなので、そういう意味では「OAuth 認証」ってのは「オレオレ Connect」みたいなもんですね。

まぁでも「オレオレ OAuth」の上で「オレオレ Connect」やる「JWT 認証」よりはマシなんじゃないかなっていう気はします。

<!-- more -->

## OpenID Connect 関連仕様、多すぎ問題

うん、OpenID Connect の仕様群は、全部読むと大変ですね。

Authlete の人も Dynamic Client Registration は読む必要無かったと思うけど、読む必要あるかどうかを判断するのがまず大変。

そして、そこは OAuth の各種 Extension を全部理解して使いこなすのも、同じように大変。

OAuth の各種 Extension を読まずに OAuth 実装して、その状態から OpenID Connect 仕様を全部実装しようとしたら、一回全部作り直しになってもしょうがないかなとは思います。

でも、自分で IdP 実装する人は、基本 [OpenID Connect Core 1.0](http://openid.net/specs/openid-connect-core-1_0.html) だけ読めば十分ですよ。それでも十分大変だけど。

RP 実装する人は、その RP が Web サイトなら [OpenID Connect Basic Client Implementer's Guide 1.0](http://openid.net/specs/openid-connect-basic-1_0.html) だけで十分。

Native アプリなら...Native アプリのケースって Backend Server 含めると 4-party ですし、3-party を想定して策定されてる OAuth や OpenID Connect の範囲を超えているので、今後の課題ですかね。（そういや [OAuth for Native Apps](http://labs.gree.jp/blog/2015/12/14831/) てのを先日書きました）

で、ちょっと話がそれましたが、結論としては、IdP 作るのは RP より大変で、Authlete みたいなの作るのは IdP 作るよりさらに大変、というだけのことなのではないでしょうか。

必要な Extension については具体的なユースケース聞かない限りなんとも言えないので、そこは Authlete 的に「とりあえず実装する」という方針だと、Authlete が大変なのはよくわかります。

一方で、ほとんどの IdP 実装は、既存 OAuth 実装に scope=openid と ID Token だけ追加実装すれば十分じゃね？とも思います。

具体的なユースケース聞かない限りなんとも言えないのですが。

## response_type 増えてる問題

これはまぁ RFC 6749 策定時には「実装はあるけどまだ標準化するレベルになかった」ものが、OpenID Connect の時代には標準化レベルに達していて、OpenID Foundation が Connect 策定するついでに [OAuth 2.0 Multiple Response Type Encoding Practices](http://openid.net/specs/oauth-v2-multiple-response-types-1_0.html) にまとめた、っていうのが実際のところ。

「現実の OAuth がとっくに RFC 6749 を超えていたのだよ」という感じでしょうか。

みんなが当時 IETF OAuth WG のスピード感に満足していれば、[OAuth 2.0 Multiple Response Type Encoding Practices](http://openid.net/specs/oauth-v2-multiple-response-types-1_0.html) は IETF 側でまとめられていたよね、というのもあるし、RFC 6749 があそこまで時間かかるんだったら、response_type=code+token は Core に入れられたよね、ってのもあるにはあるけど、いまさら言ってもしゃーないすね。

## クライアントアプリケーションのメタ情報、多すぎ問題

えっと、そもそも、Dynamic Client Registration、きっと使わないです。

Dynamic Client Registration を検討してるって人がもしいたら、まずは [OAuth PKCE](https://tools.ietf.org/html/rfc7636) ってので代用できないか検討するがいいかと思います。

Dynamic Client Registration をセキュアにやる方法は、OAuth WG と OpenID Connect の関連仕様を全部読んでもまだ足りないはず。

それでも Dynamic Client Registration したいんだ！って人は...PKI まわりの技術一通り把握したら、使いこなせる...のかな...？

という前提の元で Dynamic Client Registration で登録された Client を Public / Confidential どちらにするかという話をすると、grant_types に implicit 以外が含まれてたら Confidential ですね。

そもそも Dynamic Client Registration で Public Client 登録する意義がよくわかんないですけど。

## Unsigned JWT 作れない問題

UserInfo Response を Unsigned JWT にするってのも、きっと使わないです。

署名の無い URL-safe Base64 Encoded な JSON (= Unsigned JWT) とか、素の JSON より decode がちょっとめんどいっていう以外に、どんな意義があるというのでしょうか？

UserInfo Response を Singed and/or Encrypted JWT にするってのも、まず使わないでしょう。

「Aggregated Claims が使いたいんだ！」っていう人は検討してもいいですが、「Aggregated Claims って何？」っていう人は、きっと Aggregated Claims も使うこと無いでしょう。

## redirect_uri は必須か？

これに関しては、[Token Request Validation](http://openid.net/specs/openid-connect-core-1_0.html#TokenRequestValidation) では redirect_uri の省略を許しつつも Authentication Request の Section では REQUIRED としか書いてない Connect Core が、読みづらいですね。

Connect でも OAuth との整合性を保つため redirect_uri が1つの場合は省略可能としてもいいですが、OAuth Server のポリシーとして Authorization Request に redirect_uri を必須にしても良いです。

scope=openid の有無でそこが切り替わるっていう実装が、一番わかりづらいんじゃないでしょうか？

## さいごに

Qiita の記事を読む限り、Authlete では RP が幅広い OPTIONAL な機能を選択できるようなので、その選択で本当に大丈夫なのかの判断はなかなか高度な仕様理解を求めそうです。

どういうユースケースの場合はどういう OPTIONAL が選択された、っていうノウハウが溜まっていくと、そこは Authlete の強みになるんでしょうな。

特に Consumer 向けサービスやってる事業者さん向けには、そういうニーズ高そうな気がします。

あと、どこかのタイミングで「実際には一度も選択されたことの無い OPTIONAL 機能」とかリストで公開してもらって、それらがどういうときに使われるのかを議論するのとかおもしろそう :)