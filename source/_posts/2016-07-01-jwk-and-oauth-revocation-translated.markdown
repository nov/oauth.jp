---
layout: post
title: OAuth Revocation と JWK を翻訳しました
date: 2016-07-01 10:14
comments: true
categories:
---

どうも、この度 OpenID Foundation Japan の事務局長になった Nov です。

事務局長就任のご挨拶的なポエムを書けというオーラをふつふつと感じながら、ガン無視してこの記事を書いております。

さて、みなさん覚えておられるでしょうか？

[OpenID Foundation Japan 翻訳 Working Group](http://openid-foundation-japan.github.io) のことを。

僕が OpenID のエバンジェリストになる前からリーダーをしており、古くは 2010 年に今は亡き OpenID 2.0 の仕様を翻訳していた、あの伝説の WG を。

**その伝説の WG が、2年超の休眠期間を終えて、ついに復活します！**

復活第一弾は、OAuth Revocation と JWK！

* [OAuth 2.0 Token Revocation - RFC 7009](http://openid-foundation-japan.github.io/rfc7009.ja.html)
* [JSON Web Key (JWK) - RFC 7517](http://openid-foundation-japan.github.io/rfc7517.ja.html)

え、この2つにどういう関連性があるのかって？

特にないです！

JWK 翻訳しようとしたら、途中まで翻訳されて休眠してた OAuth Revocation の存在を思い出しただけです！

## OAuth Revocation

OAuth Revocation は、必要なくなったけどまだ有効期限が残っている Access Token や Refresh Token を、OAuth Client が OAuth Server に「もういらないよ」って伝えて、OAuth Server 側で Token の無効化を行う仕様です。

これはまぁ、これだけ実装しても Client が通知してくれなけりゃあまり意味がないので、Client にちゃんと通知させることができるようなガバナンスとセットで意味をなす仕様です。  
サポートしたい OAuth Server の中の人は、ガバナンスもがんばってください。

## JSON Web Key (JWK)

JWK は、OpenID Connect の ID Token の署名検証用の鍵を公開する場合によく使われる、JSON Format の鍵表現仕様です。

たとえば Google なんかでは、[OpenID Provider Config Endpoint](http://accounts.google.com/.well-known/openid-configuration) にアクセスすると、`jwks_uri` っていう URL が返されて、その `jwks_uri` にアクセスすると (2016.07.01 時点では) 以下のような JSON が返ってきます。

```json
{
  "keys": [{
    "kty": "RSA",
    "alg": "RS256",
    "use": "sig",
    "kid": "73d4e5a3ccd130498bdb8c47fa025464505264ff",
    "n": "yvWe5x68Dc7OV9E7v-wN6li4ChMbZM1DmIZg1C2Ei-78Mqrfl8x3tZMee-ukykEUBcYQicgBRTo8TfKjAerbSKQ8K7ZKDgzPBEGRmYI9UYpZkCwOzhJ-UYU0QB3HcbF1c0l8sZxWTbzNQkiAmEesHH7klqZWScNou2KQEZR9Cs8zHH4clFqbVp8_jVb6xXuVMkpcDGodBjPvmDHA7BI7suirtiGdAnBtZ_cAX8m6MlX3WknNQ4tr88L-XZhTu8pQi0l-uQgsnTNR0k1XWVzmhCs2ftn1kF-UC8ipnei2va0WmX4EMZ-_6rbWRunio1hr9siOFVFdwQw9m34RXiE_Ow",
    "e": "AQAB"
  }, {
    "kty": "RSA",
    "alg": "RS256",
    "use": "sig",
    "kid": "5c20e39d07fff8a69a0b134fe90bf965c8ea458e",
    "n": "r0DVakOHSFsC6x4IEGtqxERP1YJaP-tycQL5b3cpmnxvjfckHZ9pnql0TPaMHKEdjHlr68MBWatgikGLg1l8injeez_fBOk7BDGyjKezxQAY3qDiGD79CD4EuSobOhYZOiSmtRZDRSrULLtcEksOkWvoBi3aRwVPFipdOOTZvP8TRE3erp-TEtVcaACt3_rWKaW7LTA3RLsFzArVDL_tzsGMuACvz0Uab73cUSjYSS6ErJKIQ-cHqsBRhQf1aYvXxu0Jw8TxrRFwbFRgaDlt9NWptMkTAuClzs_ChXlk3K4I6m2fTaNnNgdk0I5sJel-OebertIM91SlnbGzpRSsfw",
    "e": "AQAB"
  }, {
    "kty": "RSA",
    "alg": "RS256",
    "use": "sig",
    "kid": "70f6c4267925b33136a1d1cfee4ebc57b249e5cb",
    "n": "qjpXf8YUP22n4Z5DY-YBb8ksxT0YsPDPaCfNtFyXwyE4zInlnw4dSiUud6y0WjPZaVhhVuV_jjgnOgh16lKgaJVYSEaaZDiukI02n5kZ02ZTCkqU27bafL7zBzMBssLliKgnaLFaNH8JBh0mj3suTWp0aB3hMouj1IkkdUB_MCfc6I56tyOwon5JK3vGrYk9vZ--cjTSllN9NYJcWfcUyGoI7RgNz9gvBIznD24NQR1cxmArkusaqmQj6AbARixklSiMpT1qIp0IG-L6wqFi6FHlcbUZnDxCZJVWHfCB9Gfdoox3lgBbdzAebFDomIgxpHwhxsA-iRhYyrjUlSrRiQ",
    "e": "AQAB"
  }]
}
```

これは JWK 仕様に定められた JSON Web Key Set (JWK Set) というもので、`keys` という JSON Array の中に含まれている一つ一つの JSON Object が JSON Web Key (JWK) です。

OpenID Connect の場合は RSA 鍵を使うことが多いですが、EC 鍵も Shared Secret も JWK で表現できます。

ACME や WebCrypto など、OpenID Connect 以外の仕様でも、JWK 表現された鍵を見かけることが増えてきたので、今回翻訳することにしました。

実は各暗号アルゴリズムごとに必要な JSON メンバの定義などは JSON Web Algorithm という別仕様にまとめられているので、そちらもいつか翻訳しなきゃなとは思っています。

あと JWK の識別子 (`kid`) として利用されることがおおい JWK Thumbprint という仕様も、そのうち翻訳しようと思っています。

ということで、今後も [OIDF-J 翻訳 WG](http://openid-foundation-japan.github.io) の活動をお楽しみに！

