---
layout: post
title: IETF JOSE WG と OAuth WG から一気に9本の RFC が！
date: 2015-05-20 21:05
comments: true
categories:
---

一気に出ましたね。すでに過去にもいくつかは紹介したり翻訳したりしていますが、それぞれを簡単に紹介しておきます。

### JOSE WG

* [RFC 7515](http://tools.ietf.org/html/rfc7515) - JSON Web Signature (JWS)
* [RFC 7516](http://tools.ietf.org/html/rfc7516) - JSON Web Encryption (JWE)
* [RFC 7517](http://tools.ietf.org/html/rfc7517) - JSON Web Key (JWK)
* [RFC 7518](http://tools.ietf.org/html/rfc7518) - JSON Web Algorithms (JWA)
* [RFC 7520](http://tools.ietf.org/html/rfc7520) - Examples of Protecting Content Using JSON Object Signing and Encryption (JOSE)

JWS は署名付きのデータを JSON (の Base64 URL Encode) 形式で表現するための仕様で、多くの場合は JSON Payload に対して署名するケースで利用されます。JSON じゃないデータに対して署名して、署名結果を JSON (の Base64 URL Encode) 形式で表現することもできますが...まぁ、細かい話は置いときましょう。OpenID Connect の ID Token とかは、JWS 仕様に従って署名されています。

JWE は暗号化されたデータを JSON (の Base64 URL Encode) 形式で表現するための仕様です。現状では JWS よりは利用頻度低いかとは思いますが、SAML Assertion を暗号化してるようなユースケースを Connect に移行する時なんかには使うでしょう。

JWK は JWS や JWE などで利用する鍵を JSON 形式で表現するための仕様です。上記2つとよくセットで利用されます。OpenID Connect でも、OpenID Connect Discovery をサポートしているような IdP では大体公開鍵を JWK Set 形式で公開していますね。

JWA は、JWS や JWE で利用される各アルゴリズムおよびそれらの識別子を定義している仕様です。JWT, JWS, JWE をライブラリを通じて利用しているケースでは、あまり気にすることはないでしょうが、JOSE ライブラリ作者は読むことになるでしょう。

最後のは、サンプルリストですね。これはまぁライブラリ作者が読むくらいでしょう。

### OAuth WG

* [RFC 7519](http://tools.ietf.org/html/rfc7519) - JSON Web Token (JWT)
* [RFC 7521](http://tools.ietf.org/html/rfc7521) - Assertion Framework for OAuth 2.0 Client Authentication and Authorization Grants
* [RFC 7522](http://tools.ietf.org/html/rfc7522) - Security Assertion Markup Language (SAML) 2.0 Profile for OAuth 2.0 Client Authentication and Authorization Grants
* [RFC 7523](http://tools.ietf.org/html/rfc7523) - JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants

JWT は JSON (の Base64 URL Encode) 形式で Assertion を生成するための仕様です。OpenID Connect の ID Token などで利用されています。大抵は署名 (= JWS) とセットで利用されるでしょう。

Assertion Framework for OAuth 2.0 (ry) は、任意の Assertion を OAuth 2.0 の Client Authentication で Client Credentials として使ったり、Authorization Grant として利用して Assertion を Access Token と交換するための仕様です。これ単体では利用できず、後の2つのサブ仕様の共通部分を抽象化した仕様になっています。

SAML 2.0 Profile for OAuth 2.0 (ry) は、SAML Assertion を RFC 7521 の Assertion として利用するための仕様です。既存の SAML SP が持っている SAML Assertion を OAuth 2.0 の Access Token と交換して API Access させたいとか、そういう時に使います。「あぁ、それは SAML 単体じゃ無理なんで、ID-WSF 必要ですねぇ〜」って言われたら、多分 ID-WSF 無視してこれ使えば、OAuth 2.0 使えるようになるはずです。

JWT Profile for OAuth 2.0 (ry) は、JWT を RFC 7521 の Assertion として利用するための仕様です。Client Authentication 目的で利用するケースは、ADFS / Azure AD とかであるはずですが、Authorization Grant として利用するケースは...あったかな？

### Links

* [JSON Web Token (JWT) - OAuth.jp](/blog/2012/10/26/json-web-token-jwt/)
  * もうこの記事書いてから、2年半経ってるんですね〜。感慨深い。
* [JWS 実装時に作りがちな脆弱性パターン - OAuth.jp](/blog/2015/03/16/common-jws-implementation-vulnerability/)
  * すでに JOSE ライブラリは大体出揃ってるので、今後新しく作ることはあまりないかもしれませんが、もし自分で JWS 実装する時は、これ注意してくださいね。
* [翻訳ドキュメント一覧 - OpenID Foundation Japan 翻訳 WG](http://openid-foundation-japan.github.io)
  * Draft 版の JOSE 仕様群は、こちらに翻訳版あります。
* [JWSとJWTがRFCになりました！ - @_Nat Zone](http://www.sakimura.org/2015/05/2997/)
  * Nat さんはじめ、みなさんおつかれさまでした！
