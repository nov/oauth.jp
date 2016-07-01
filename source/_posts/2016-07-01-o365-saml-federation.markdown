---
layout: post
title: Office 365 と外部 SAML IdP との連携設定
date: 2016-07-01 11:03
comments: true
categories:
---

どうも、事務局長の Nov です。

どうも、ジムキョクチョのノブです。

どうも、ノブキョクチョです。

どうも、のぶチョです。

そう、のぶ千代です。

最近歳のせいか、Rails で SAML IdP とか作ってます。

今日は自作 SAML IdP を Office 365 と連携させてみたので、その格闘の記録を残しておきます。

Office 365 の制約とか Azure AD の制約とか全く前提知識なしに格闘した記録なんで、そういうのいいから手っ取り早くやり方教えろやって人は我らがふぁらおぅ兄さんのこちらの連載をご覧ください。

* [Office365/AzureAD - OpenAMとのID連携 (1)](http://idmlab.eidentity.jp/2014/11/office365azureadopenamid.html)
* [Office365/AzureAD - OpenAMとのID連携 (2)](http://idmlab.eidentity.jp/2014/12/office365azureadopenamid.html)
* [Office365/AzureAD - OpenAMとのID連携 (3)](http://idmlab.eidentity.jp/2014/12/office365azureadopenamid_25.html)

<!-- more -->

## Office 365 & Azure AD のドメインが違う問題

僕、最初に Azure AD に YAuth.jp のディレクトリ (`yauth.onmicrosoft.com`) 作成して、そのあと別の機会に Office 365 の Subscription (`yauthjp.onmicrosoft.com`) を開始したんで、それら2つが別ディレクトリになっておりまして、前者に `yauth.jp` ドメインを紐付けてたんで、Office 365 側に `yauth.jp` ドメインを紐付けられなかったんですね。

で、二つを Merge しようとして [Azureサブスクリプションの既定のディレクトリをO365で利用しているAzure Active Directoryに変更する方法](http://ebi.dyndns.biz/windowsadmin/2016/04/07/azure%E3%82%B5%E3%83%96%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%97%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E6%97%A2%E5%AE%9A%E3%81%AE%E3%83%87%E3%82%A3%E3%83%AC%E3%82%AF%E3%83%88%E3%83%AA%E3%82%92o365%E3%81%A7/) ってのを教えてもらったんで、やってみようとしたんですが...

よくみたら Merge じゃなかったっていうね。

で、まぁ Azure AD 側の `yauth.onmicrosoft.com` はもうあきらめて消しちゃえってことで消そうとしたら、ユーザーいるから消せねーだの、アプリが登録されてるから消せねーだの言われて、ちまちま消した挙句、`Office 365 Management APIs` ってのが最後どうしても消せず...

こうなった。

![Azure AD Directories](/images/posts/azure/azure-ad-directories.png)

消せないけどいらん `yauth.onmicrosoft.com` 側のディレクトリを "remove-me" にリネームして、そっちから `yauth.jp` の紐付け解除して `yauthjp.onmicrosoft.com` 側のディレクトリに紐付け直した。

しょっぱなから解決してないですね。

でも結論から言いますと、これ、たぶん SAML 連携とは全く関係なかったんだろうなって、いまでは思います。

## PowerShell 必須問題

なんか Azure AD の管理画面には SAML IdP 追加する箇所ないんですよ。

SAML IdP の登録に必要なパラメータなんてせいぜい5-6個やのに、PowerShell ってやつが必須なんですよ。

で、どうも Google 先生がいうには、Mac では PowerShell 使えないっていうじゃないですか。

で、立ち上げましたよ、Windows 10 の VM。  
Mac から Remote Desktop 経由でいじりたかったんでね。

また Surface Pro4 使う機会逃しましたよ。

そしてまた Windows 10 に最初っから入ってる PowerShell じゃ、Azure AD には繋げないんですよ、これが。

いろいろ Google 先生に聞いて出てくるやつかたっぱしから試しても、なんかずっと謎のエラーでつづけてて、あぁ〜、これが MS 流のエラーってやつかぁとか思いながら、最終的にはこれにしたがってエラーなくなりました。

[Azure Active Directory Cmdlets - Install the Azure AD Module](https://technet.microsoft.com/en-ca/library/jj151815.aspx#bkmk_installmodule)

でもここももう一回ゼロからやり直せって言われたら、またはまる自信ありまくり。

うん、ここもまた解決してない。

## PowerShell 経由で SAML IdP 登録

これは [Office365/AzureAD - OpenAMとのID連携 (2)](http://idmlab.eidentity.jp/2014/12/office365azureadopenamid.html) にしたがって `Set-MsolDomainAuthentication` っての走らせたらすんなり完了。

## Federated Domain は Primary にできない制約

しらんかったよ、Federated Domain は Primary にできないなんて。

MS さん、`Failed to change primary domain.` しか言ってくれないしさ。

Primary Domain にできないと、そのドメインのメアド持ったユーザー作れないじゃないですか。

SAML IdP 登録したのに Federation できないじゃないですか。

なんか無駄に DNS 設定変えてみたりして、DNS 浸透待ちしまくったよ。

結果、Google 先生に「Federated Domain は Primary にできない」って教えられたよ。

## User の Immutable ID 61文字までしか入れられない制約

Federated Domain は Primary にできないんで、Azure AD 管理画面からは Federated Domain のユーザーを追加できないんですね。(いや、別にできてもいいとおもうねんけど、なんかそういう制約があるんです)

つまり、ここでも PowerShell 必要なんですよ。

そして、Google 先生に導かれたのがこの PowerShell コマンドです。

[New-MsolUser](https://technet.microsoft.com/en-ca/library/dn194096)

でもまた謎のエラーで、しばらく経ってからやり直せとか言われる。

どんだけまっても同じエラーで、これはしばらく経ってもラチあかねんんじゃね？ってなって...

結果、Google 先生に「Immutable ID には61文字までしか入れられない」って教えられたよ。

こっちは64文字だよ...

うん、とりあえず SHA1 取った。

解決...なのか？w

## SAML Request スカスカやのに SAML Response への制約は多い

SAML Response に関する制約はこちらにまとまってる通りで、それはまぁ SAML IdP 自作できてればそんな対応大変じゃないんですけども...

[シングル サインオンを実装するための SAML 2.0 ID プロバイダーの使用](https://msdn.microsoft.com/ja-jp/library/azure/dn641269.aspx)

SAML Request スカスカなくせに Response に対する制約多くないすか？

SAML Request に `ProtocolBinding` とかつけてくれないんすか？ `AssertionConsumerServiceURL` は？

まぁその辺 OAuth でも省略しても動いたりすることありますけど、そこ省略するとライブラリとかが大変ですよね。

いや、ライブラリそんなないし、O365 対応できないライブラリなんてニーズないんでしょうから、別にいいんですけどね。

でもまぁ、その辺って、センスですよね。

## おわりに

と、まぁなんだかんだあったけども、無事 O365 にログインできました！

大人の階段のぼってるわぁ〜

プロ千代なってまうわぁ〜