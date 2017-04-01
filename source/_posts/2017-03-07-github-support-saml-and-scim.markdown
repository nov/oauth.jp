---
layout: post
title: GitHub Business Plan 登場、GitHub.com ドメインで SCIM と SAML をサポート。
date: 2017-03-07 21:18
comments: true
categories:
---

オンプレサーバーで GitHub Enterprise をお使いのみなさま。

日々オンプレサーバーのメンテ、おつかれさまです。

GitHub.com の Business Plan っての、良さそうですよね。

SCIM と SAML サポートしてて、いままでオンプレ版でしかできなかった Provisioning と Federation (SSO といった方が伝わるか？) が、GitHub.com ドメインでできるようになったんですよ。

これで GitHub.com が生きてればいつでも Deploy できますよ。

死ぬもんね、オンプレ。てか、一旦死んだら結構しばらくの間死ぬもんね、オンプレ。

GitHub.com なら、AWS が生き返ればきっと生き返るよ。

ということで、Business Plan の SAML と SCIM、ちょっと試してみましたよ。

# SAML 設定

GitHub Help の [この一連のドキュメント](https://help.github.com/articles/managing-member-identity-and-access-in-your-organization-with-saml-single-sign-on/) を読む限り、SAML 設定しないと SCIM 使えないようです。

あ、SAML の SP Entity ID とかはドキュメントには書いてないです。

Business Plan 契約して設定画面行かないと、SP Entity ID も Assertion Consumer Service (ACS) URL もわかんないです。

まず金払ってからしか試せません。

**ファーストひどい。**

<!-- more -->

で、金払って Organization の Security 設定ページ (https://github.com/organizations/YOUR-ORGANIZATION/settings/security) にアクセスすると、SAML 設定が ON にできるようになってます。

ここに以下の情報が表示されています。

* Organization single sign-on URL
  * https://github.com/orgs/YOUR-ORGANIZATION/sso
* Assertion consumer service URL
  * https://github.com/orgs/YOUR-ORGANIZATION/saml/consume

あれ？SP Entity ID は？と思ったそこのあなた。

SP Entity ID はここには記載されていません。

**セカンドひどい。**

実際 SAML Request を発行させてみないとわからないっぽいんですが、面倒なんでここに書いときますね。

* SP Entity ID
  * https://github.com/orgs/YOUR-ORGANIZATION

です。

で、これらを元に IdP 側に SP 登録して、その後 GitHub 側で以下の情報を入力すると、SAML 設定自体は完了です。

* Sign on URL
* Issuer (IdP Entity ID)
* Public Certificate (IdP Certificate)

え？NameID Format はって？

えぇ、特にドキュメントにも設定画面にも指定ないですね。

そう、特にドキュメントにも設定画面にも指定ないです。

**サードひどい。**

Business Plan になってからサポートに問い合わせたら、Gist に書かれたドキュメントが送られて来て、そこには NameID Format は Email だよって書いてあったんで、Email 推奨のようです。

が、実は UUID とか指定しても動きます。

あと、Email 指定してもそのメアドのユーザーにログインできるわけでもないです。

nov@example.com の NameID を持つ SAML Assertion で、nov@matake.jp の GitHub アカウントにログインできたりします。

SAML Federation とか言うてるけど、結局既存の GitHub.com アカウントに SAML IdP 側のアカウントを Link するだけなんですよ。

その紐付けは、SAML での初回ログイン時に GitHub.com 側の ID & Password 入力して行います。

つまり、nov@example.com の NameID を持つ SAML Assertion をどの GitHub アカウントに紐づけるかは、エンドユーザーが決定権を持っています。

情シスじゃないです。

**フォースひどい。てかお前それ致命的やろ。**

ちなみに、NameID は不変かつ同一 IdP 内でユニークでさえあればなんでもいいです。

Email 推奨してたドキュメントは、多分なんか気分的に Email のがいんじゃね？くらいのノリだったんだと思います。

実際には NameID Format が Persistent で NameID が UUID、とかにしても普通に動いてました。

## SCIM 設定

SCIM にいたっては、一切のドキュメントが公開されていません。

GitHub が公式サポートをうたう OneLogin から GitHub.com を選んで設定してみましたが、OneLogin 側に設定が必要な SCIM Base URL の値すらどこにも書いてありません。

**もうひどいカウントやめていいですか？**

はい、サポートに問い合わせて聞いたら Gist のドキュメント送られて来ます。

普通に SCIM Base URL 書いてあります。

* SCIM Base URL
  * https://api.github.com/scim/v2/organizations/YOUR-ORGANIZATION

で、ここに `admin:org` scope 持つ OAuth2 Token を添えて SCIM API Request 送ってやればいいです。

あ、どの属性が必須かって？

Gist ドキュメントもらえば書いて...ないですけど、API リクエスト送り続けてエラー見続けたらそのうちわかります。

ということで、現状の挙動見る限り、必須な SCIM User Attributes は以下の通りです。

* id
* userName (email)
* name
  * familyName
  * givenName
* emails (userName と同じメアドを primary に)

はい、ドキュメントさえあれば、プロビジョニングなんて簡単ですよ。

で、プロビするじゃないですか？

さすがにプロビしたアカウントに関しては、SAML で NameID にそのメアド指定したら GitHub.com アカウントとの紐付けとかなくするっとログインできると思うじゃないですか？

はい、SAML 初回ログイン時にアカウント紐付け必須です。

てかそもそも SAML Assertion に指定されるメアドと GitHub.com アカウントのメアドが一致する必要はないので、SCIM でプロビしたアカウントに別メアドの SAML Assertion 送りつけて紐付けたりもできます。

**お前それ致命的やろ。**

よくそれで月額 **$21/user** も取れると思ったな。

いや、Business Plan アカウントに対するサポートからの回答速度は爆速ですけど。

1時間もしないうちに回答きますけど。

そこはすばらしいですけど。

**お前それ致命的やろ。**

Octocat はかわいいけども。

<a href="https://www.amazon.co.jp/Github-Octocat-Figurine-%E3%82%AA%E3%82%AF%E3%83%88%E3%82%AD%E3%83%A3%E3%83%83%E3%83%88-%E3%83%95%E3%82%A3%E3%82%AE%E3%83%A5%E3%82%A2/dp/B01L6VR96O/ref=as_li_ss_il?ie=UTF8&qid=1491031942&sr=8-7&keywords=github&linkCode=li3&tag=bianca0b-22&linkId=0a81153b8af09233c305993da8a2a3ee" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=B01L6VR96O&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=bianca0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=bianca0b-22&l=li3&o=9&a=B01L6VR96O" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />