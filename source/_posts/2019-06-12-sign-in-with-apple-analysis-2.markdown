---
layout: post
title: "Sign in With Apple の特徴分析 (2)"
date: 2019-06-12 15:14
comments: true
categories:
---

[Sign in With Apple の特徴分析 (1)](/blog/2019/06/08/sign-in-with-apple-analysis/) のつづき。

文章でつらつら書いても結局わかりづらいもんはわかりづらいんで、もう箇条書きで淡々と。

これだけそろってれば、あと最後の ***要調査*** ってとこを各自必要に応じて調べるなり、誰かがこの記事にコメントしてくれるなりすれば、一通りみなさんのサービスの設計に落とせるんでは無いかと思います。

※ 落とせない人は [僕 (= YAuth.jp 合同会社) に仕事を依頼する](https://yauth.jp) という手もありますw

<!-- more -->

## 各種 ID とその対象 (?) 範囲

### Developer が使う IDs 等

* Team ID
  * 個人向け Apple Developer Account では Developer Account に1つ
  * Enterprise 向け Apple Developer Account での話はしらない
* App ID
  * App Store に並べるアプリ毎に1つ
  * MacOS App と iOS App では別 App ID
  * Native SDK で Sign in with Apple する場合の Client ID に相当
  * App ID に対して Sign in with Apple の設定をする際には以下が必要
    * Primary App ID の指定
  * Sign in with Apple の設定を通じて Primary App ID に紐づく
* Service ID
  * App Store に並べるアプリには紐づかない ID
  * 現状この Service ID を使ってできることは Sign in with Apple のみ
  * Service ID に対して Sign in with Apple の設定をする際には以下が必要
    * Primary App ID の指定
    * Web Domain の設定と Domain Verification
    * Return URIs (= redirect_uri) の指定
      * 複数可
      * 上記 Verified Domain 外の URL でも指定可
        * Domain Verification の存在意義は不明..
  * Native SDK の外で Sign in with Apple する場合の Client ID に相当
  * Sign in with Apple の設定を通じて Primary App ID に紐づく
* Primary App ID
  * App ID および Service ID に対して Sign in with Apple の設定をする際に指定
  * Primary App ID 単位で Key を発行
  * Primary App ID 単位で Client Access を Revoke
  * (たぶん) Primary App ID 単位で Private Email を発行
* Key
  * Client Secret に指定する JWT に署名するための EC 秘密鍵
  * Primary App ID に紐づく
  * 1つの Primary App ID に複数の Key を発行可能
* Client Secret に指定する JWT
  * iss
    * Team ID
  * aud
    * Apple IdP の Issuer Identifier
  * sub
    * Native SDK で code を受け取った場合は当該アプリの App ID
    * Native SDK 以外で code を受け取った場合は当該サービスの Service ID
    * code を access_token と交換する役目を負う Backend Server の Client ID とは限らない点に注意
  * iat
    * 現在時刻 (UNIX Timestamp)
  * exp
    * 現在より未来かつ6ヶ月以内の時刻 (UNIX Timestamp)
    * 通常は Token Request 毎に動的に JWT を生成し数分後くらいの exp を指定するものと思われる
  * 署名鍵
    * 当該 Client に紐づく Primary App ID に紐づいた Key
* Private Email Relay Service 設定
  * Team ID に紐づく
  * 当該 Team ID 配下の全 App ID / Service ID に対して発行された Private Email Address にメール送信できる送信元ドメイン & メールアドレスを指定
  * 10 ドメインまでしか登録できない
    * 10+ サービスを同一 Team ID 配下に登録しつつサービス毎に送信元ドメイン変えるとかは無理かも

## User に紐づく IDs 等

* ID Token の sub
  * Team ID 単位で異なる PPID
* Private Email
  * 現状これがどの単位で異なる値となるのか不明 (実機の iOS 13 beta も Simulator も挙動が不安定)
    * Team ID 単位？
    * Primary App ID 単位？ <= Revoke 単位見る限りたぶんこれ
    * App ID 単位？
    * Primary App ID 単位？
  * Revoke 毎に変わる
    * sub は Revoke 前後でも変わらないんだしメアドだけ変わる意味は無いのだが...
  * 当該アドレスに紐づく Team ID の Private Email Relay Service に登録された送信元ドメイン & メールアドレスからしかメールが届かない
    * Revoke 画面で Revoke はせずにメール転送を OFF にすることも可能
  * 単一 Team ID 内に複数の Primary App ID がある場合は単一 sub に複数 Private Email を紐づけて保存する必要あり
    * sub は Team ID 単位でメアドは Primary App ID 単位であることからくる現象
      * ※ あくまで Private Email が Primary App ID 単位で発行されるという前提で話をしている
    * Email の転送 ON/OFF 設定は Revocation 単位 (= Primary App ID 単位) なので、転送 OFF したつもりが別アプリ向けメアドにメール来たりすると問題
* Client Access の Revocation
  * Primary App ID 単位

## その他調べるべき項目

* Touch ID / Face ID 使えるパターン・使えないパターン
  * iOS
    * SDK (ASAuthorizationAppleID~) => 使える
    * ASWebAuthenticationSession => ***要調査***
    * SFAuthenticationSession => ***要調査***
    * SFSafariViewController => ***要調査***
    * WKWebView => ***要調査***
    * UIWebView => ***要調査*** でもまだこれいるの？
  * MacOS
    * SDK (ASAuthorizationAppleID~) => ***要調査*** たぶん使える
    * ASWebAuthenticationSession => ***要調査***
    * WebView => ***要調査*** そもそも MacOS の場合の WebView とかさっぱりわからん
  * Safari
    * w/ Apple JS SDK => 使える
    * w/o Apple JS SDK => 使えない
      * ただし近い将来 appleid.apple.com ドメイン全体で Touch ID / Face ID が使えるようになる兆しあり
  * Android => 使えない
  * Safari 以外のブラウザ => 使えない

ということで、要調査のところはそろそろめんどくさくなってきたんでわかった人教えていただけるとありがたいですw
