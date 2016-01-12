---
layout: post
title: OAuth IdP Mix-Up Attack とは？
date: 2016-01-12 15:51
comments: true
categories:
---

成人式を迎えられたID厨の皆様におかれましては、大変おめでとうございます。成人前からID厨とか、キモカワですね。

さて、今日は、成人式直前にOAuth MLに投下された以下のpostで「OAuth 2.0の脆弱性」として紹介されている "IdP Mix-Up Attack" について紹介します。

[[OAUTH-WG] OAuth Security Advisory: Authorization Server Mix-Up](https://mailarchive.ietf.org/arch/msg/oauth/JIVxFBGsJBVtm7ljwJhPUm3Fr-w)

## 前提条件

* End-UserとRPの間の (TLS-protectedでない) HTTP Request/ResponseをAttackerがproxy可能。
* RPが2つ以上のOAuth Server (IdP) と接続しており、そのいずれかが攻撃者の管理下にある。(IdPの中の人が実は攻撃者だった etc.)
* RPは複数のIdPに対して共通のredirect_uriを利用しており、そこへのcallbackを受け取った時にstate値を元にIdPを特定する。
  * ただしIdP側のredirect_uri検証が「事前登録済の値との完全一致」でない場合は、この通りではない。

## 攻撃フロー

以下、Attackerの管理下にあるIdPをAIdP (Attacker IdP)、その他の悪意ないIdPをHIdP (Honest IdP) と呼ぶことにします。

![OAuth IdP Mix-Up Attack](/images/posts/oauth-idp-mixup.png)

1. End-UserはRPの任意のページから「HIdPでログイン」ボタンをクリック。
2. Browser->RPへの (TLS-protectedでない) リクエストをProxyしたAttackerは、RPに「AIdPでログイン」するリクエストを送信し、AIdPのAuthorization EndpointへのRedirect Responseを受け取る。
3. AttackerはBrowserにHIdPのAuthorization EndpointへのRedirect Responseを返す。ただしstate値は2で受け取った「AIdP向けのAuthorization Requestとひもづいた」値を利用する。(これ以降全リクエストはTLS-protectedであるためAttackerのProxyは介入不可)
4. End-UserはHIdPでApproveボタンをクリック。
5. HIdPはstateとcode/tokenをquery/fragmentにつけた状態で、End-UserをRPのredirect_uriに戻す。
6. code/tokenを受け取ったRPは、state値を元に受け取ったAuthorization ResponseがAIdPからのものだと判断する。
7. RPはAIdPのToken EndpointやAPI Endpointにcodeやtokenを送りつけてしまい、HIdPのcodeやtokenがAIdP (攻撃者) の手に渡る。

## 対策方法

### 対策方法1

AIdP向けのredirect_uriとHIdP向けのredirect_uriを分ける。ただしこれが効果を発揮するためには、HIdPがRPからAIdP向けのredirect_uriを受け取った際にエラーになる必要がある。

### 対策方法2

Authorization Responseにcode/token発行者のIdentity情報を含める。

いまOAuth WGで提案されている対応策は、以下の2つです。なんか似たようなのが2つ出てきててカオスですね。

* http://tools.ietf.org/html/draft-jones-oauth-mix-up-mitigation
* http://tools.ietf.org/html/draft-sakimura-oauth-meta

### 対策方法3

悪意のある可能性があるIdPを採用しない。(まぁ、普通は採用してないですよね...)

### どの対応策を採用すべき？

RPは以上の3つのうちいづれかを採用すれば良いですが、IdPが採用できるのは2しかないですね。

## Malicious Endpoint Attackってのは？

[OAuth 2.0 Mix-Up Mitigation - draft-jones-oauth-mix-up-mitigation-00](http://tools.ietf.org/html/draft-jones-oauth-mix-up-mitigation) には、"IdP Mix-Up Attack" とは別に "Malicious Endpoint Attack" ってのが出てきます。

これは、Discovery & Dynamic Client Registrationを前提とした状況で発生する攻撃パターンです。

Dynamic Registrationを前提とすると、「信頼できないIdPを採用しない」という選択肢が取りづらくなるため、より問題が大きくなります。

が、そもそもみなさんそんなDynamic Client Registrationとか使ってないでしょうし、これについての解説はまた機会があればということで...

## 最後に

ID厨で本当に今年成人式を迎えられたあなた！次回の [#idcon](http://idcon.org) の懇親会タダにするんでご容赦くださいm_ _m