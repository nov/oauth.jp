---
layout: post
title: "Azure AD B2C が外部 API 向けに払い出す JWT-formatted Access Token について"
date: 2017-03-24 17:08
comments: true
categories:
---

[Azure AD B2C Access Tokens now in public preview](https://azure.microsoft.com/ja-jp/blog/azure-ad-b2c-access-tokens-now-in-public-preview/)

ということで、さわって見ました。

## Step.1 Azure AD B2C テナントの作成

まずは Azure AD B2C テナントを作成します。なんか portal.azure.com から行くと Classic Portal ベースのドキュメントに飛ばされるので、新しい方の Portal ベースのドキュメントをリンクしときますね。

[Azure Active Directory B2C: Create an Azure AD B2C tenant](https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-get-started)

テナントを Subscription と紐づけるとかいう処理は Production Use でない限り Skip で OK です。

## Step.2 Web API (Resource Server) および Client の登録

[Azure AD B2C Access Tokens now in public preview](https://azure.microsoft.com/ja-jp/blog/azure-ad-b2c-access-tokens-now-in-public-preview/) にしたがって、Web API と Client を登録します。

Web API の登録には Scope の定義もセットです。

Client の登録には Key (Client Secret) の作成と利用可能な API & Scope の設定がセットです。

ちなみに、Resource Server と Client がどちらも同じメニューから作成し、横並びで表示されるのはちょっと違和感ありますね。

Resource Server に Redirect URI (Reply URL) の登録が必須だったり、Implicit を使えるようにするかの選択は Client 側の設定項目で Resource Server 側にそれ設定してもなんの意味もなかったりというのは、MS さんらしいというかなんというか。

なお、

* App ID URI を登録した Application は固有の scope を定義できるようになり、Resource Server になれる。
* App ID URI を登録しない Application は Key (Client Secret) しか作成できず、Client にしかなれない。
* Resource Server は Key (Client Secret) も作成できるので同時に Client にもなれる。

というルールになっているようですが、3つめの Resource Server かつ Client というのは同じ aud を持つ Access Token と ID Token が生成されることに繋がるので、よほどの事情がない限りやめるべきです。

<!-- more -->

## Step.4 Policy の作成

Azure AD B2C に外部 Resource Server 向けの JWT 形式の Access Token を払い出させるには、Policy ID を指定しないといけません。

理由は不明ですが、MS さんがそういうのだから仕方ない。

ということで、なんか適当に Sign-in Policies というところに Policy を登録します。

Policy 登録終わると「Metadata Endpoint for this policy」とかいう URL が表示され、そこにアクセスすると OpenID Connect Provider Configuration Document (JSON) が得られます。

OpenID Connect Discovery では Query Parameter とかサポートしてないはずですが、Policy ID (p=xxx) が Query についてるのは MS さんの悪い癖で、なんかやめられない感じなんでしょう。

この Policy は Resource Server ごとに指定するものでも無いようなんで、まぁデフォルトのまま放置して Policy ID だけ取得しとけば OK です。

## Step.3 Access Token の取得

この Gist に適宜自分で作った Client ID やらなんやら指定してやれば、HTTP のやりとりなどが Console に表示されるんで、それ見ながらフロー確認してください。

[azure_ad_b2c_without_credentials.rb](https://gist.github.com/nov/0673c8ad02e23a875f05b2be43dd040a)

また、もろもろの Azure AD B2C 環境セットアップがめんどくさいという方の為に、こちらに僕が作った RS & Client を Client Secret 含め置いておきます。

[azure_ad_b2c.rb](https://gist.github.com/nov/9e9b537b897fa2585e085ab1b83b2e3d)

動作確認用のユーザーも作っておきましたので、これが正常に動いてる間はご自由にお使いください。

Username: you@sts4b2c.onmicrosoft.com
Password: ]6]yzxXYG7uruM4p

動かなくなったら、あきらめて自分で Azure AD B2C 環境セットアップしてください。

## わかったこと

* Access Token の audience は Resource Server Application の Object ID。
* Access Token の audience は Array にはできない。
  * 複数 Resource Server にまたがった Access Token は発行できず、そのような Scope の指定の仕方した時点でエラーになります。
  * これはセキュリティ的には良いことですね。
* ID Token と Access Token の署名鍵は同じ。
  * これはお行儀悪いですね。
  * Resource Server が Client を兼ねた場合、署名鍵や audience だけではその JWT が Access Token なのか ID Token なのか区別できないケースが発生し、脆弱性につながりかねません。
* "openid" 以外の Scope を指定しないと Token Response に access_token が含まれない。
  * [RFC 6749](https://tools.ietf.org/html/rfc6749#section-4.2.2) に違反しており、利用しているライブラリによってはユーザーが Query 書き換えただけで Client 側で 500 エラー発生させたりできそうです。

ということで、ここで発行された Access Token を受け取った Resource Server は、

* 署名が正しく
* JWT の "aud" Claim が自身の Azure AD B2C 上の Object ID であり
* 必要な Scope が "scp" Claim に含まれており
* "azp" Claim に含まれる Client ID が正当な Client のものであり
  * 余談ですが同じく MS さんが主導の [OAuth 2.0 Token Exchange](https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-07#section-4.3) では "cid" Claim なのに、ここでは "azp" なのはなんというか...w
* 期限切れでない

ことを確認する感じになりますね。

今日のところは以上です。