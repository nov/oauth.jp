---
layout: post
title: "OAuth2 in Action が Amazon にも"
date: 2017-04-01 16:42
comments: true
categories:
---

我らが Justin Richer がずっと執筆進めてた [OAuth2 in Action が Amazon.co.jp に登場](http://amzn.to/2nVbgRV)してました。

目次はこんな感じ。

* Part 1 - First steps
  * What is OAuth 2.0 and why should you care?
  * The OAuth dance
* Part 2 - Building an OAuth 2 environment
  * Building a simple OAuth client
  * Building a simple OAuth protected resource
  * Building a simple OAuth authorization server
  * OAuth 2.0 in the real world
* Part 3 - OAuth 2 implementation and vulnerabilities
  * Common client vulnerabilities
  * Common protected resources vulnerabilities
  * Common authorization server vulnerabilities
  * Common OAuth token vulnerabilities
* Part 4 - Taking OAuth further
  * OAuth tokens
  * Dynamic client registration
  * User authentication with OAuth 2.0
  * Protocols and profiles using OAuth 2.0
  * Beyond bearer tokens
  * Summary and conclusions

<!-- more -->

まだ読んでないんですが、目次見る限り Part2 までは初心者向け、Part3 はセキュリティ厨の皆様向け、Part4 は ID 厨の皆様向けって感じでしょうか。

ちなみに Justin は、先日翻訳した [NIST SP 800-63C](/blog/2016/07/15/nist-800-63c/) の英語版の著者の1人でもあり、Java の OpenID Connect 実装の1つである [MITREid Connect](https://id.mitre.org/connect/) の中の人でもあります。

Part4 の OAuth Dynamic Client Registration に関しては、以下の2つの RFC の Editor です。

* [OAuth 2.0 Dynamic Client Registration Protocol](https://tools.ietf.org/html/rfc7591)
* [OAuth 2.0 Dynamic Client Registration Management Protocol](https://tools.ietf.org/html/rfc7592)

いままでの OAuth 2.0 関連の本よりだいぶガチな感じ。

OAuth の本見ると、一度書き始めて途中で挫折した過去 (OAuth1.0 の部分書き終えたとこで放置してるw) が蘇りますが、もぅこれ翻訳すりゃいんじゃね？っていうね。

<a href="https://www.amazon.co.jp/OAuth-2-Action-Justin-Richer/dp/161729327X/ref=as_li_ss_il?ie=UTF8&qid=1491032403&sr=8-3&keywords=oauth&linkCode=li3&tag=bianca0b-22&linkId=99872d3ee518929dc6ee8ce757316cab" target="_blank"><img border="0" src="//ws-fe.amazon-adsystem.com/widgets/q?_encoding=UTF8&ASIN=161729327X&Format=_SL250_&ID=AsinImage&MarketPlace=JP&ServiceVersion=20070822&WS=1&tag=bianca0b-22" ></a><img src="https://ir-jp.amazon-adsystem.com/e/ir?t=bianca0b-22&l=li3&o=9&a=161729327X" width="1" height="1" border="0" alt="" style="border:none !important; margin:0px !important;" />