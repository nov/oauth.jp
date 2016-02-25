---
layout: post
title: "「OAuth 認証」を定義しよう"
date: 2016-02-25 11:27
comments: true
categories:
---

「OAuth 認証」って言葉が出てくると、「認証と認可は違う」とか言い出す人が出てきて、大体の場合「OAuth 認証」言ってた人たちがやりたいことの話とはズレた議論が始まるので、もういっその事「OAuth 認証」とは何かを定義してみましょうかね。

## 「OAuth 認証」で Relying Party (RP) がやりたかったこと

RP (OAuth Client) は、ブラウザの前にいる人を、認証したかったんですよね？

もう少し正確にいうと、ブラウザの前にいる Entity が、RP 側で把握しているどの Identity と紐付いているか、というのを知りたかったんですよね？

いきなり Entity とか Identity とかいう専門用語が出てきてアレですが、そのあたりのことは先日の OpenID TechNight #13 でもお話ししたので、以下のスライドの Entity・Identity・Authentication・Authorization のあたりのページを見てください。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/Lw5OsZp5n6qwXb" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>

で、あるサービスが End-User を認証したいって思った時って、普通は当該サービスが事前に当該 Entity に対して ID と Password を登録させて Identity Record (「サービスアカウント」と言ってもいい) を作成し、後日当該 Entity が当該サービスに訪れた際は登録済みの ID と Password を提示させて当該 Entity に紐づく Identity を確定するわけです。

でも Identity Federation (上記スライドでは「ID 連携」と表現している) という手法を使うと、外部のサービス (Identity Provider, IdP) の Identity を当該サービス (Relying Party, RP)の Identity と紐付けて管理することで、End-User から直接パスワードを預からなくても、ブラウザの前にいる Entity (= End-User) に紐づく IdP 側の Identity に紐付いた RP 側の Identity を確定できるので、結果として End-User に紐づく RP 側の Identity を確定することができます。

で、どうやって IdP と RP がそれぞれに管理している Identity を紐付けるのか、というのが、Federation Protocol の肝なわけですが、「OAuth 認証」というのはその一種です。

<!-- more -->

## 「OAuth 認証」とは

OAuth 自体は「End-User が OAuth Server に管理を委託している Resource に対して、OAuth Client がアクセスすることを許可する方法」を標準化した OAuth Core (RFC 6749) と、「End-User の許可を得た OAuth Client が実際に OAuth Server 上の Resource にアクセスする方法」を標準化した OAuth Bearer (RFC 6750) を核とした標準仕様群なので、Federation Protocol に必要な「IdP 上の Identity をどう表現し RP に伝えるか、という仕組み」は持っていません。

しかし、「IdP 上の Identity をどう表現し RP に伝えるか、という仕組み」は別に標準化されていなくてもいいんですよ。どうせみなさんほとんど Facebook Login しかしないんだから。

っていうことで、「IdP (OAuth Server) 上の Identity 情報にアクセスする API へのアクセス権だけを OAuth Protocol に従って RP (OAuth Client) に渡しましょう。Identity 情報にアクセスする API は、まぁ JSON に user_id とか含めりゃだいたい動くでしょ。」っていうノリでできあがったのが、「OAuth 認証」です。

標準化された方法ではないけども、Identity Federation はしてますね。

Identity 情報にアクセスする API って各 IdP で仕様バラバラなんですけど、みんな OAuth 使ってアクセス権やり取りするってところは共通です。

実際には Identity Federation に必要な多くの部分がそのバラバラな部分に依存したりしてるわけで、とても標準的な Federation Protocol とは呼べない代物ですけども。

ということで、「OAuth 認証」の定義としては、

<b>「RP が、IdP の提供する任意の Identity 情報取得 API に対して OAuth Protocol に従ってアクセスすることで、Identity Federation を行う方法の総称」</b>

とかでいかがでしょうか？

ちなみに、OpenID Connect においては、UserInfo API ってのが標準化された Identity 情報取得 API で、Identity 情報に加えて IdP 側の認証セッション情報をやり取りする手段が ID Token です。

Identity Federation に必要な情報の多くの部分は実は IdP 側の認証セッションに関する情報だったりするわけですが、その辺の話は、また今度にしましょう。

## 補足) 結局「認証か認可か」という議論は何だったのか？

さて、OAuth は「End-User が OAuth Server に管理を委託している Resource に対して、OAuth Client がアクセスすることを許可する方法」でしたね。

「End-User が OAuth Client 上の Resource へのアクセス権を持つかどうかを、OAuth Client が判断する方法」ではなくて。

普通「認証か認可か」について議論するときって、以下のように主語と目的語が揃っていて、動詞だけが違う条件の元で議論すると思うんですよ。

* IdP authenticates End-User
* IdP authorizes End-User's access (to Resource on IdP)

でも「OAuth を利用した認証」と「OAuth における認可」っていう比較をする時って、以下の2つを比較してませんか？

* RP authenticates End-User (relying on IdP's help)
* End-User authorizes RP's access (to Resource on IdP) / IdP authorizes RP's access (to Resource on IdP, on behalf of End-User)

いや、それ比較してもいいんですよ。いいんですけど、よほどの前提知識がない限り、主語と目的語と動詞が全部違う2つのコンテキストを比較して、意味のある結論を導き出すことは困難なんじゃないかと思うんです。

本当に「認証か認可か」って比較をするのであれば、以下のコンテキストで比較して意義ある答えにたどり着くようなコンテキストで議論したほうがいいよなぁ、と。

* Service X authenticates End-User Y
* Service X authorizes End-User Y's access (to Resource Z)

なので、「OAuth と OpenID Connect」の比較の流れで出てくる「認証と認可」っていう議論に関しては、生暖かく見守っております :p