---
layout: post
title: OAuth / Connect における CSRF Attack の新パターン
date: 2016-05-06 14:19
comments: true
categories:
---

昨日こんなのが OAuth ML に流れてました。

[[OAUTH-WG] Another CSRF attack](https://mailarchive.ietf.org/arch/search/?email_list=oauth&gbt=1&index=G4J3H1BMyIN01FCOLKqLjrx7AZ4)

## 前提条件

RP (Relying Party a.k.a. OAuth Client) が2つ以上の IdP (Identity Provider a.k.a. OAuth Server) と連携している状況で、片方の IdP に悪意がある。

* 悪意ある IdP = AIdP (A は Attacker の略)
* その他の IdP = HIdP (H は Honest の略)

## 攻撃フロー

![Another CSRF Attack](https://www.websequencediagrams.com/cgi-bin/cdraw?lz=YWx0IFZpY3RpbSdzIFNlc3Npb24KICAgAA4HLT5SUDogTG9naW4gdy8gQUlkUAAZBVJQLT4AMQY6ACENQUlkUDogQXV0aFogUmVxIHcvIHN0YXRlLVYAUAVBSWQALwoAIgVlbnRpY2F0ZQA0EgAmC0gAUAZ0dGFja2VyJ3MgY3JlZGVudGlhbHMAgSgFSABTBQB3BmNvZABIBgBfDkhJZFAncwAZBQCBCwsgKHRvIHJlZGlyZWN0X3VyaSBmb3IASgUpAIF2EACBbwlSUDoANglpcyB0aWVkIHRvIHRoZSBzAII7BiwgT0shAIIfCQCBOAYAgQ4JAIEmBlIAgUMOYWNjZXNzX3Rva2VuICYgaWQABQYAglwQIEhlbGxvLCB3aGF0J3MgeW91ciBuYW1lPwCDIhFJJ20Ag1EHCmVsc2UAgjEMAINaDACCTAgAg0wcAIJwCACDCwcAKQkAgw8HAINaEkEAgwgMADUIAINcEgA7DwAgFQCDIQ4AUwYAgScNAIMDE0EAgi1pAIFTCldlbGNvbWUgYmFjaywAhkoHIQplbmQK&s=earth)

1. Victim が AIdP を使って RP へのログインを試みる。
2. RP は Authorization Request を AIdP に送る。
    * AuthZ Req には Browser Session と紐付いた `state` パラメータをつけている。
3. AIdP は Victim を認証し、必要に応じて同意を取得する (ふりをする)
    * 同時に AIdP は裏で HIdP から `code` を取得する。
    * HIdP の code は Attacker の HIdP 上のアカウントに紐付いているもの。
4. AIdP は HIdP から取得した `code` を、RP の HIdP 用 `redirect_uri` に返す。
    * この時 Step2 で RP が発行した `state` を付与する。
5. RP は Browser Session と紐付いた `state` を検証した上で `code` を HIdP の Token Endpoint に送る。
6. HIdP は Attacker のアカウントに紐付いた `access_token` を返す。
7. RP は Victim をログインさせる。
    * ここで HIdP 上の Attacker アカウントと RP 上の Victim アカウントが紐づけられる。

そして、Attacker は任意のタイミングで自身の HIdP 上のアカウントを使って、RP 上の Victim のアカウントにログインできる。

## 防御策

`state` をリクエスト先の IdP とも紐づけましょう。