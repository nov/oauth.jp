---
layout: post
title: "Zero-day in Sign in with Apple Deep-dive"
date: 2020-06-02 12:22
comments: true
categories:
---

先週末に [Zero-day in Sign in with Apple](https://bhavukjain.com/blog/2020/05/30/zeroday-signin-with-apple/) とかいう記事が出ていました。

この記事ではいまいち詳細がわからないんで、ちょっと実際 Apple IdP の挙動調べてみたら、当該 Endpoint 発見しました。

### 実際にどこが脆弱だったのか

1. 非 Safari ブラウザで、適当な RP にアクセス
    * e.g.,) http://signin-with-apple.herokuapp.com/
    * Safari だと OS の Native UI が出てきてブラウザ遷移しないので注意
    * 既に連携済の RP の場合は一度連携解除しておくこと
2. Sign in with Apple のボタンをクリックして Apple AuthZ Endpoint に遷移
    * AuthZ EP: https://appleid.apple.com/auth/authorize
3. ログイン後、初回連携時にのみ出てくる同意画面 (メアド選択するところ) に到達
    * ![Sign in with Apple Consent Screen](/images/posts/apple/siwa-consent.png =500x)
4. ここで「続ける」を押すと内部的に Ajax Call が実行される
    * Ajax API EP: https://appleid.apple.com/appleauth/auth/oauth/authorize
    * 選択されたメアドやその他の AuthZ Req Params が一通り JSON で POST される
    * レスポンスとしては `id_token` とか `code` なんかが含まれた JSON が返ってくる
    * ![Sign in with Apple Consent Ajax Submit](/images/posts/apple/siwa-consent-ajax-submit.png =500x)

で、問題は Step.4 で Submit されるメールアドレスが、実際当該 Apple ID に紐づいてないものでも OK だったということですね。

<!-- more -->

[The Real Cause of the Sign In with Apple Zero-Day](https://aaronparecki.com/2020/05/31/30/the-real-cause-of-the-sign-in-with-apple-zero-day) という解説記事では、メールアドレスの値そのものを POST するんではなく `real_email` / `proxy_email` という Flag 値のみを POST すべしとか書いてますが、実際には Apple ID には複数のメールアドレスが紐付き Sign in with Apple の同意画面でのメールアドレス選択肢も3つ以上になるケースがあるので、メールアドレスの値そのものを POST することは理にかなっています。

問題は、POST されたメールアドレスが当該アカウントと紐づいてるかを Apple IdP サーバーがチェックしてなかったという点だけですね。

現在はメールアドレスとアカウントの紐付けがサーバーサイドでチェックされているので、変なメールアドレスを投げるとちゃんと 400 が返ってきます。

### Email 以外を書き換えるとどうなる？

ちなみにこの Ajax Call のタイミングで `client_id` & `redirect_uri` を通信途中で改竄してやると、別 Client 向けの Code が正規 Client の正規 Redirect URI に返されるなんていう挙動もあります。

Submit された値に基づいてサーバーサイドで Auto-submit Form なりをレンダリングしてレスポンスを返すのではなく、既に同意画面中に仕込まれた JS とパラメーター値を使ってレスポンス返してるので、Ajax Call 中で書き換えられた `email` 以外の値はレスポンスを返すべきか否かの判断には使われていないのですね。

まぁそれ自体がそこまで大きな問題かというと、そんな気はしませんが。。。ちょっと設計が微妙というか、フロントエンジニアが勝手に設計したらこうなっちゃったみたいな雰囲気が満載ですね。

ということで、まだ ID Token の `aud` は別の値にされる可能性は残っているのですが、まぁ普通に `code` 使ってバックチャネルで ID Token 取れば、`aud` が書き換えられた ID Token は発行されないんで、そうすればよいです。

Sign in with Apple のフロントチャネルで発行された ID Token は、ガン無視して捨ててやれば良いのです。あれにはまず使い道はありません。

### 余談 : OAuth 2.0 WMRM 使われてる！

Sign in with Apple には Popup Mode なるものがあります。

実際以下の URL にアクセスして黒い方のボタンをクリックするとそれが体験できます。  
http://signin-with-apple.herokuapp.com/?popup=true

ここ、よく見ると OAuth 2.0 Web Message Response Mode (WMRM, `response_mode=web_message`) が使われています。

大昔、[@zigorou](https://twitter.com/zigorou) さんと [@_nat](https://twitter.com/_nat) さんとノリで Independent  Draft まで書いて、その後放置してるやつですね。懐かしいですねぇ.. #遠い目

[OAuth 2.0 Web Message Response Mode](https://tools.ietf.org/html/draft-sakimura-oauth-wmrm-00)

Apple さんは `redirect_uri` に Origin 指定したりとかはしてないんですけどね。

OAuth WMRM、実は Auth0 とか Okta とかも使ってるんですよね。Okta のはなんか変な Prefix ついてるけど。

* [OAuth 2.0 Authorization Framework - Auth0](https://auth0.com/docs/protocols/oauth2#how-response-mode-works)
* [OpenID Connect & OAuth 2.0 API - Okta](https://developer.okta.com/docs/reference/api/oidc/#postmessage-data-object)