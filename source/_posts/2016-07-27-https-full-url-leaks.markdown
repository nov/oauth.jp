---
layout: post
title: "HTTPS でも Full URL が漏れる？OAuth の code も漏れるんじゃね？？"
date: 2016-07-27 10:48
comments: true
categories:
---

なんですかこれは！

[New attack bypasses HTTPS protection on Macs, Windows, and Linux](http://arstechnica.com/security/2016/07/new-attack-that-cripples-https-crypto-works-on-macs-windows-and-linux/)

DHCP につなぐと PAC ファイルがダウンロードされて HTTPS であろうとアクセス先の Full URL は漏れるですって？

Web Proxy Autodiscovery ですって？

チョットニホンゴデオネガイシマス

ってことで、まぁこれが実際どれくらい簡単に実現できる攻撃パターンなのかは他のセキュリティ業界の方々に後で聞くとして、この記事でも触れられてる OpenID Connect とか OAuth2 への影響について、ちょっとまとめておきましょうか。

## Authorization Request & Response が漏れる

`response_mode=form_post` なんていうのも一部ありますが、基本 OAuth2 / OpenID Connect の Authorization Request & Response は GET です。

Implicit Flow の場合は Response Parameter が URL Fragment についてるので、Server に送られる Full URL が漏れたところで特に URL Fragment の内容は漏れないですが、Code Flow の場合は Response の Query についてる Authorization Code は漏れますね。

まぁ Authorization Request に含まれる `state` とかも漏れますが、今回のケースだと Cookie とかは漏れないんで、`state` が漏れること自体は大して問題ではないでしょう。

ただ、Redirect URL に HTTPS 使っても `code` 漏れるってのは、辛そうですね。

## Authorization Code が漏れたらどうなるの？

Code 置換攻撃 (Code Cut & Paste Attack) が可能になります。

ここに漏れた `code` があるとしましょう。すると、以下の手順で攻撃者は `code` 所有者の RP 上のアカウントにログインすることができます。

1. 攻撃者自身のブラウザで Authorization Request 発行
2. 攻撃者自身の IdP 上のアカウントで IdP にログイン
3. Authorization Response を途中で止める
4. Authorization Response 中に含まれる `code` を被害者のものに置換 (`state` は置換しない)
5. Code 置換済の Authorization Response を RP に送る

こうすると、RP が `state` を使った CSRF 対策を行っていても、RP は受け取った `code` を使って攻撃者を被害者としてログインさせてしまいます。

これが Code 置換攻撃です。

では、これを防ぐ手立てはあるのでしょうか？

<!-- more -->

## OpenID Connect の場合 : ID Token の nonce をチェック

OpenID Connect では、Authorization Request で `nonce` というパラメータを送ることができますね。

あれは `state` とよく似た役割を果たしますが、`code` とは紐付かない `state` と異なり、`nonce` は `code` と紐づいて保存され、最終的に発行される ID Token に含まれて返ってきます。

よって、`code` だけ置換しても、`nonce` に紐付いた Cookie なりを奪わない限り、RP が Token Endpoint から返ってきた `nonce` をチェックした時点で `code` 置換を検知できることになります。

ようするに、OpenID Connect の仕様的には OPTIONAL やけど、とりあえず `nonce` 使っとけや、ってことですね。

## OAuth 2.0 の場合 (1) : PKCE 拡張を使う

`nonce` のない OAuth 2.0 の場合、Authorization Request & Response のセッションと `code` を紐付けるパラメータが特にありません。

そして、それでは Code 置換攻撃は防げません。

よって、`code` と紐付いたパラメータを用意してやる必要があります。

OpenID Foundation Japan 事務局長としては「OpenID Connect 使えや」って話でもあるわけですが、もうちょっとお手軽な方法としては [OAuth PKCE](https://tools.ietf.org/html/rfc7636) というのもあります。

PKCE はもともと `client_secret` を持てない OAuth Client 向けに作られた仕様ですが、Authorization Request で送った `code_challenge` と紐づく `code_verifier` を Token Endpoint に送ることになるので、当然ながら IdP 側では `code` と `code_challenge` を紐付けて管理することになります。

つまり、`code` を置換すると、`code_verifier` が合わなくなって、Token Request が失敗する、と。

ちなみに、PKCE には `code_challenge_mode` っていうパラメータがありますが、`code_challenge_mode=plain` は Authorization Request みれる状況では `code_verifier` 自体が漏れることになるんでダメで、この攻撃防ぐためには `S256` を使ってくださいね。

さて、これで OAuth 2.0 でも Code 置換攻撃、防げましたね。

これからは Confidential Client (`client_secret` 持てる OAuth Client) でも PKCE 使えってことですね。

ただ PKCE は OAuth 拡張なんで、OAuth Server (IdP) 側がまず PKCE 対応する必要があります。

Facebook Login とか使ってる人たちは、どうすればいいんでしょうねぇ〜

## OAuth 2.0 の場合 (2) : response_mode=form_post を使う

PKCE 同様 `response_mode=form_post` も拡張仕様なのでどの OAuth Server (IdP) でも使えるわけではないですが、`response_mode=form_post` を使うと Authorization Code は POST Body に含まれて返されるので、Full URL が漏れても `code` は漏れません。

Session と紐付けた `nonce` なり `code_verifier` なりを管理するより、RP にとっては `response_mode=form_post` を使うほうが楽かもしれませんね。

「Session と紐付ける」って概念、意外に通じないことも多いですし。

## まとめ

OpenID Connect を使ってる場合は、RP が常に `nonce` 使えばいいだけなんで、RP だけが注意してれば大丈夫ですね。

OAuth 2.0 を使ってる場合は、OAuth Server (IdP) 側がまず PKCE なり OpenID Connect なりに対応して、RP がそれを使う必要がありそうです。

**Facebook のみなさん、聞こえますかぁ〜**

ps.

「OAuth 2.0 の場合」というセクションタイトルになってますが、OpenID Connect でも nonce の代わりに PKCE なり response_mode=form_post 使ってもいいです。

OAuth 2.0 のレベルで解決できてるなら、OpenID Connect 特有のパラメータを利用しなくても (この攻撃は) 防げます。