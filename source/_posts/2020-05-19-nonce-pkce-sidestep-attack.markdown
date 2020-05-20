---
layout: post
title: "Nonce/PKCE Sidestep Attack"
date: 2020-05-19 13:06
comments: true
categories:
---

OAuth 2.1 で PKCE 必須にするしない議論が盛り上がっておりますが、今日そのスレで新しい攻撃パターンが議題に上がりました。  
https://mailarchive.ietf.org/arch/msg/oauth/HZt851AobQlJMBVd6_p8iWSyRS8/

今のところ "Nonce/PKCE Sidestep Attack (仮" と名付けられたこの攻撃は、以下のようなシナリオで成立します。

### Nonce/PKCE Sidestep Attack (仮

#### [前提条件]

ある Client に、アプリケーションコンテキストによって PKCE を使うコンテキストと Nonce を使うコンテキストが共存している。

#### [攻撃シナリオ]

以下、[原文](https://danielfett.de/2020/05/16/pkce-vs-nonce-equivalent-or-not/#noncepkce-sidestep-attack) を要約。

1. 攻撃者は何らかの手段により Nonce に紐づいた (= PKCE code_challenge とは紐づいていない) 被害者の Code を詐取する。
2. 攻撃者は PKCE を使うコンテキストにおいて自らのブラウザで Client に Authorization Request を実行させ、code_challenge だけを抜き去ったものを Authorization Server に送る。
3. 攻撃者は受け取った Authorization Response 中の Code を Step.1 で取得した Code と差し替え Client に提示する。
4. Client は受け取った Code を Step.2 で生成した code_challenge に紐づいた code_verifier を付与して Token Request を実行する。
5. Authorization Server は code_challenge と紐づいていない Code を受け取ったので code_verifier は無視して Token Response を返す。
6. Client は Nonce を使うコンテキストでもないので Nonce のチェックも行わず、code_verifier が無視されたことも検知できないので成功裏に Token Response を受け取ってしまう。

以上により Code 置換攻撃が成立する。

<!-- more -->

しかし、ここでひとつ疑問があります。

Authorization Server は、本当に code_challenge と紐づいていない Code を受け取った場合、それに添えられた code_verifier を無視するのでしょうか？

もし無視するとしたら、以下のような攻撃も成立してしまうのではないでしょうか？

### PKCE CSRF Protection Bypass (仮

#### [前提条件]

ある Client は CSRF 攻撃対策として state ではなく PKCE を使っている。

#### [攻撃シナリオ]

1. 攻撃者は自身のブラウザで Client に Authorization Request を実行させ、code_challenge だけを抜き去ったものを Authorization Server に送る。
2. 攻撃者は受け取った Authorization Response を一旦保存する。
3. 攻撃者はフィッシングサイト等を介して被害者のブラウザで被害者に気づかれない形 (hidden iframe etc.) で Authorization Request を実行させつつ Step.2 で受け取った Authorization Response を被害者のブラウザに渡す。
4. 被害者から Code を受け取った Client は、被害者のブラウザセッションに紐づいた code_verifier を付与して攻撃者の Code を Authorization Server に提示する。
5. Authorization Server は code_challenge と紐づいていない Code を受け取ったので code_verifier は無視して Token Response を返す。
6. Client は code_verifier が無視されたことも検知できないので成功裏に Token Response を受け取ってしまう。

SameSite=Lax がデフォルトでないようなブラウザでは、上記のような攻撃は成立してしまうでしょう。

また以下のようなパターンもありえるかもしれません。

### PKCE Code Injection Protection Bypass (仮

#### [前提条件]

ある Client は CSRF 攻撃対策としては state を、Code Injection 攻撃対策としては PKCE を使っている。

また Client は redirect_uri が HTTP であったりするなど、Code が漏洩しうる条件も持っている。

#### [攻撃シナリオ]

1. 攻撃者は自身のブラウザで Client に Authorization Request を実行させ、code_challenge だけを抜き去ったものを Authorization Server に送る。
2. 攻撃者は Authorization Response を受け取り Code を一旦保存する。
3. 攻撃者は被害者の Code が詐取できるポイントで、被害者の Code を詐取する代わりにそれを Step.2 で受け取った攻撃者の Code に置換する。
4. 被害者から Code を受け取った Client は、被害者のブラウザセッションに紐づいた code_verifier を付与して攻撃者の Code を Authorization Server に提示する。
5. Authorization Server は code_challenge と紐づいていない Code を受け取ったので code_verifier は無視して Token Response を返す。
6. Client は code_verifier が無視されたことも検知できないので成功裏に Token Response を受け取ってしまう。

う〜ん... `Authorization Server は code_challenge と紐づいていない Code を受け取ったので code_verifier は無視して Token Response を返す。` っていう挙動を許してしまうと、PKCE で対策されていたはずだった CSRF 攻撃も Code Injection 攻撃も防げなくなってしまいますね...

まぁそもそも被害者のブラウザ上で hidden iframe で Authorization Request を実行させたり Code を置換できたりするようなケースはほとんどないとは思いますが...

PKCE それでいんだっけ？？

ということで、過去の僕の実装を見てみると...

[Rack::OAuth2 Server-side PKCE Implementation](https://github.com/nov/rack-oauth2/blob/master/lib/rack/oauth2/server/extension/pkce.rb#L28)

やっぱ code_challenge に紐づいてない Code と code_verifier が渡されてきたら、ちゃんとエラーにしてますね。

じゃあ PKCE 採用 Authorization Server として最もメジャーな Google さんはどうでしょう...

[Google PKCE Client Sample](https://gist.github.com/nov/9feba86685bd3b18b4bf7bfb88022046)

やっぱ code_challenge に紐づいてない Code と code_verifier が渡されてきたら、ちゃんとエラーにしてますね。

う〜ん、このケースでの挙動、[RFC 7636 - Proof Key for Code Exchange by OAuth Public Clients](https://tools.ietf.org/html/rfc7636) では特に触れられてないんですけど、やっぱ普通に考えれば、code_challenge に紐づいてない Code と code_verifier が渡されてきたら、エラーですよね...

みなさんの実装はいかがでしょうか？

### 追記

上記 "PKCE CSRF Protection Bypass (仮" に関しては、以下の記事では "Downgrade Attack on PKCE" と名付けられたようです。  
https://danielfett.de/2020/05/16/pkce-vs-nonce-equivalent-or-not/#downgrade-attack-on-pkce