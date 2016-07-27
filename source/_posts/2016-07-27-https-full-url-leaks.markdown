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
4. Authorization Response 中に含まれる `code` を被害者のものに置換 (state は置換しない)
5. Code 置換済の Authorization Response を RP に送る

こうすると、RP が `state` を使った CSRF 対策を行っていても、RP は受け取った `code` を使って攻撃者を被害者としてログインさせてしまいます。

これが Code 置換攻撃です。

では、これを防ぐ手立てはあるのでしょうか？

<!-- more -->

## OpenID Connect の場合 : ID Token の nonce をチェック

OpenID Connect では、Authorization Request で `nonce` というパラメータを送ることができますね。

あれは `state` とよく似た役割を果たしますが、`code` とは紐付かない `state` と異なり、`nonce` は `code` と紐づいて保存され、最終的に発行される ID Token に含まれて返ってきます。

