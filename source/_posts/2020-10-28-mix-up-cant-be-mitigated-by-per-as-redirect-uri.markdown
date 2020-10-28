---
layout: post
title: "Mix-up Attack は Per-AS redirect_uri では防げない"
date: 2020-10-28 23:05
comments: true
categories:
---

いままで Mix-up Attack は Client が AS 毎に redirect_uri を使い分けていれば防げると信じられてきましたが、それじゃ防げないケースもあるよってのが [OAuth ML に投稿](https://mailarchive.ietf.org/arch/msg/oauth/RjbSwFRmLsk0EgAY2Ter-nw66EY/)されました。

細かい解説は英語読んでもらうとして、シーケンスにするとこういうことです。

![OAuth IdP Mix-Up Attack rev.2](/images/posts/oauth-idp-mixup-rev2.png)

Attacker AS が (Display Name やロゴ等を通じて) 一見 Honest Client に見えるような Client (Attacker Client) を Honest AS に登録しておく必要があります。

User が Attacker AS 選んでるのに Honest AS に飛んで Approve してしまってる部分も、[Attacker Proxy](/blog/2016/01/12/oauth-idp-mix-up-attack/) が利用可能な状況 (e.g., Client が HTTP なエンドポイントで Honest AS のログインボタン等を表示している) であればもっと違和感のないフローになるでしょう。

で、解決策として AuthZ Response に "iss" を含めることが提案されているのですが、「[悪意のある可能性があるIdPを採用しない](/blog/2016/01/12/oauth-idp-mix-up-attack/)」ってのが一番の解決策であることに代わりはありません。