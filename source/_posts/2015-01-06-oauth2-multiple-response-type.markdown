---
layout: post
title: "OAuth 2.0 の Response Type 全パターン"
date: 2015-01-06 20:15
comments: true
categories:
---

特に新しい話題ではないですが、定期的に質問されてる気がするので記事にしときます。

OAuth 2.0 の Core には、"code" と "token" という2つの response_type が定義されています。

それぞれ "Code Grant" と "Implicit Grant" と呼ばれることもありますし、歴史的経緯により "Code Flow" と "Implicit Flow" と呼ぶこともあります。

ほとんどのケースでは、この2つの response_type のどちらかを使っているかと思いますが、実はこれ以外にも以下の response_type のパターンが存在します。
([仕様はこちら](http://openid.net/specs/oauth-v2-multiple-response-types-1_0.html))

* none
* code token
* id_token
* id_token code
* id_token token
* id_token code token

id_token ってのが含まれてるのは、OpenID Connect で利用する response_type なので、OAuth 2.0 だけを実装する場合は無視して OK です。

でもこんなにいっぱいあると、Server 側 (OP 側) もどこまで実装したらいいかわかんないし、Client 側 (RP 側) もどれ使うのが一番いいのかわかんないですよね。特に OpenID Connect 使う場合は。

そんなこんなで、「どれ実装したらいいの？」とか、「どれ使えばいいの？」っていう質問を、Server 側の人にも Client 側の人にも、半年に一回くらいされる気がします。

この機会に、それぞれの response_type で想定される Use-Case をまとめておきましょう。

<!-- more -->

### none

「Approve はするけど、access_token (と id_token) は今はいらない」って時に使います。

はい、「それいつやねん」って思ったそこのあなた、あなたはきっとこれ使う事ないです。

世の中のごく一部には、これが必要な人もいるようですが、僕もいままで使ったことないです。

### code token

Native App で access_token 使うんだけど、Native App の Backend Server 側でも access_token (と id_token) が欲しい時に使います。

fragment に code と access_token がついて返って来るので、access_token はそのまま Native App が内部に保持して、code だけを Native App から Backend Server に渡します。

Backend Server はその code を Token Endpoint に送って access_token (と id_token) を取得します。

もしかしたら Native App から access_token を Backend Server に送るような実装してる人もいるかもしれませんが、そうすると以下のようなデメリットがあります。

* Token Replace Attack に対して脆弱になりがち
* Backend Server でも refresh_token を取得できない
* Implicit Flow 経由で発行される access_token は lifetime が短くなりがち (Server 側の実装依存)

### id_token

access_token が不要な時に使います。

Web App / Native App どちらでも使われうるかと思います。

ただしこれ使うと fragment に id_token ついて返ってくるので、Web App からは扱いづらいかもしれません。

「response_type=code 使って access_token 捨てればいいやん」って言われたら、「その通りですね」とお返しします。

また、Native App 側で id_token 使ってユーザー認証するケースって、そんなに多くはなさそうです。

この response_type は、あまり使う機会なさそうです。

### id_token code

Native App で id_token 使うんだけど、Native App の　Backend Server 側でも access_token (と id_token) が欲しい時に使います。

fragment に code と id_token ついて返ってくるので、id_token はそのまま Native App が内部に保持して、code だけを Native App から Backend Server に渡します。

Backend Server はその code を Token Endpoint に送って access_token (と id_token) を取得します。

ただこの response_type も、Native App 側で id_token が必要ってケースなんで、あまり使う機会なさそうです。

※ なぜか YConnect だけはこれを response_type=code 相当の response_type として利用しないといけないのですが、あれは YConnect の Bug です。

### id_token token

Backend Server に code 渡す必要なくて、Native App 側だけで id_token と access_token 使えればそれで OK ってときに使います。

fragment に access_token と id_token ついて返ってきます。

この response_type も、Native App 側で id_token が必要ってケースなんで、あまり使う機会なさそうです。

### id_token code token

Native App 側でも Backend Server 側でも id_token と access_token 両方必要な時に使います。

fragment に access_token と code と id_token ついて返ってくるので、access_token と　id_token はそのまま Native App が内部に保持して、code だけを Native App から Backend Server に渡します。

Backend Server はその code を Token Endpoint に送って access_token (と id_token) を取得します。

この response_type も、Native App 側で id_token が必要ってケースなんで、あまり使う機会なさそうです。

### まとめ

Server 側は、とりあえず以下の3つさえサポートしてれば、大抵の Use Case はサポートできそうです。

* code
* token
* code token

Client 側も、基本は以下のような基準で response_type 選べば良さそうです。

* Web App なら code
* Backend Server なしの Native App なら token
* Backend Server ありの Native App なら code token

### 補足

本記事全体として、Native App を JS App に置き換えても同様です。

また、広く公開される IdP を作る場合は、以下のようなケースも考慮しましょう。

#### Native App 側で id_token が必要なケース

Native App 側でマルチアカウント対応してるような場合は、Native App 側で id_token が必要になるケースはあるかもしれません。

そういう Client にも対応したい Server は、(none 以外) 全部サポートしましょう。

#### Native App 側で code を使うケース

Native App から Token Endpoint に code を送るケースもあります。

この場合は、Native App 側で refresh_token を取得したいケースが多いです。

access_token の lifetime が短く、Native App で頻繁に AuthZ Request を送りたくない場合などには、Native App でも refresh_token が必要になるのです。

ただしこの場合、Token Endpoint での Client Authentication を OPTIONAL にする必要があるので注意してください。