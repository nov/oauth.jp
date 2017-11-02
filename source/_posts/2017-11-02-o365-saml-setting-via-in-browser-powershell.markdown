---
layout: post
title: Azure Portal 上の In-Browser PowerShell から O365 SAML 設定"
date: 2017-11-02 15:22
comments: true
categories:
---

どうも、MS MVP (Enterprise Mobility) の Nov です。

普段もっぱら O365 の SAML 設定をいじって、自作 SAML IdP と Federation する毎日です。

いや、年に6回くらいかな。

で、毎回 PowerShell の使い方忘れるので、メモ代わりに過去にもこんな記事書いてきました。

* [Office 365 と外部 SAML IdP との連携設定](/blog/2016/07/01/o365-saml-federation/)
* [PowerShell for AzureAD](/blog/2017/05/18/powershell-for-azuread/)

そんな、普段はもっぱら Azure 上の Windows 10 VM から PowerShell いじってる僕ですが...

今日気づいてしまったんです！！[Azure Portal](portal.azure.com) 上で In-Browser PowerShell が動くようになってるってことに！！

<!-- more -->

![In-Browser PowerShell](/images/posts/azure/powershell-in-browser.png)

「Mac 上の Safari で開いた Azure Portal で Bash なんか動かしてどうすんねん」思ってた僕ですが、動くのが PowerShell となれば話は別です。

もう Windows いらんやん。

ということで、早速 O365 の SAML 設定を In-Browser PowerShell からやってみました。

まずは必要な PowerShell Module をインストールして... (初回のみ)

```ps1
# 初回のみ
Install-Module MSOnline, AzureAD
```

Module Import & AzureAD に管理者アカウントでログイン (PowerShell 起動毎に一度だけ)

```ps1
Import-Module MSOnline, AzureAD
$credential = Get-Credential
Connect-MsolService -Credential $credential
```

そして SAML 設定。

```ps1
$certificate = "MIIDVzCCAj...(中略)...QQBsHQ=="
$entity_id = "https://nov-idp.dev"
$logout_url = "https://nov-idp.dev/saml2/o365/logout"
$login_url = "https://nov-idp.dev/saml2/o365/login"
$brand_name = "OAuth.jp (O365)"
$domain = "oauth.jp"

Set-MsolDomainAuthentication -DomainName $domain -FederationBrandName $brand_name -Authentication Federated -PassiveLogOnUri $login_url -SigningCertificate $certificate -IssuerUri $entity_id -LogOffUri $logout_url -PreferredAuthenticationProtocol SAMLP
```

すべて Safari さえあれば OK です。

もう Windows いらんやん。

時代は Surface Pro LTE モデルだっていうのに。

<a href="https://www.amazon.co.jp/%E3%83%9E%E3%82%A4%E3%82%AF%E3%83%AD%E3%82%BD%E3%83%95%E3%83%88-Surface-%E3%83%8E%E3%83%BC%E3%83%88%E3%83%91%E3%82%BD%E3%82%B3%E3%83%B3-Office-FJT-00014/dp/B071P77272/ref=as_li_ss_il?ie=UTF8&qid=1509604915&sr=8-1&keywords=surface+pro&&linkCode=li3&tag=bianca0b-22&linkId=d10ccff459fa49956b8bfd0bbf07592f" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=B071P77272&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=bianca0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=bianca0b-22&l=li3&o=9&a=B071P77272" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />

ps.

なぜか `Connect-MsolService` 単体では動きません。

Windows 10 上では `Connect-MsolService` ってやるとブラウザポップアップ開くのに、なぜかブラウザ内で `Connect-MsolService` するとブラウザが立ち上がらないのです。

不思議なこともあるものです。