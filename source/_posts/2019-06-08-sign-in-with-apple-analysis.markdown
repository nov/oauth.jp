---
layout: post
title: "Sign in with Apple 特徴 (1)"
date: 2019-06-08 21:04
comments: true
categories:
---

[前記事](https://oauth.jp/blog/2019/06/04/sign-in-with-apple/) で書いたように、ここ数日 Sign in with Apple 用の RubyGem 作りながら、Sign in with Apple の特徴というか、他の IdP との違いみたいなところいろいろ調査したので、現時点での Sign in with Apple に対する雑感をまとめておきます。

## Client ID と Team ID および App ID との関係

個人として Apple Developer Account 使ったことしかないんで、会社として Developer 登録してる時の Team の扱いとかよくわかってないんですが、Apple Developer Account 登録すると Team ID ってのが割り振られます。個人だと 1 Developer Account に 1 Team ID。

この1つの Team ID の下に、複数の子 App ID が登録可能です。1 iOS App についき 1 App ID。

さらに、App ID には "Primary" という概念もあり、1つの Primary App ID の下に複数の App ID を登録することができます。

ここまでは、お仕事で実際に iOS App 作ってる人たちにはきっと当たり前な話なんだと思います。

そして、Primary App ID の下に App ID の代わりに Service ID っていうのが登録できます。

Service ID っての設定画面には Sign in with Apple 関連の設定項目しかないから、これは今回初めて出てきたものなのかもしれませんね。

で、iOS App で Sign in with Apple 使う場合は App ID というのが OAuth Client に該当し、Web とか Android とかで Sign in with Apple 使う時は、 Service ID というのが OAuth Client に該当するようです。

1 Team ID の下に複数 App ID が登録され、1 App ID の下に複数の子 App ID と Service ID が登録され、iOS/Mac Native の世界では App ID が、それ以外では Service ID が OAuth Client となる。

ここまではいいですね？いや、正直間違ってるかもしれんけどw

そして、Client の鍵 (Client Secret を生成する為の秘密鍵、詳しくは後述) を別途生成するのですが、この鍵は Primary App ID 毎に登録することになります。

鍵は Key Rotation のことも考慮してか、1 Primary App ID に複数同時に登録できます。

ただ、1 OAuth Client に 1 Private Key、という単位ではないので、鍵管理のやり方は普通の OAuth Client の場合と異なるやり方が必要になる Developer さんもいるかもしれませんね。

<!-- more -->

## User ID の発行ルール

Apple が発行する User ID (Subject Identifier) は PPID (Pairwise Pseudonymous Identifier) になっています。

通常の OIDC では、PPID は同一ユーザーでも Client ID が異なれば異なる値となり、複数 Client 間でユーザーの名寄せを防ぐ効果があります。

しかし Apple の場合は Web では Web の Service ID、iOS App では iOS App の App ID というように、同一サービスでも Client ID が異なるので、それらの間でユーザーの名寄せができないと困ってしまいます。

そのため、Apple は Team ID 単位で PPID を発行します。

同一ユーザーでも、User ID が渡されるアプリの Team ID が違えば User ID は異なるが、同一 Team ID 内では全ての App ID および Service ID にまたがって同一の User ID を受け取ることができます。

Primary App ID 単位で PPID 払い出す方が自然な気がしますが、まぁその辺は Apple Developer 経験無いんでよくわかりません。

## Email アドレスの発行ルール

Native SDK 経由で Sign in with Apple 使った場合は、"email" scope を要求するとユーザーの Email アドレスを受け取ることができます。(現状では Web では同じ scope 要求しても Email を受け取る術はありません)

この Email アドレスは、当該 Apple ID に紐づいたメアド (複数存在することあり) を渡すか、Apple が発行するランダムなメアドを渡すかを、ユーザーが選ぶことができます。

ランダムなメアドを使った場合、そのメアドは当該 Client のアクセスを Revoke した時点で利用不可能となり、次回同一ユーザーが再度 Approve すると別のメアドが発行されることになります。

通常の Social Login では、初回登録時にメアドを取得したら、次回以降のログイン時には別メアドが渡されてきても無視していることが多いと思いますが、Sign in with Apple でランダムなメアド受け取っている場合には、次回以降のログイン時に別のランダムなメアド渡されると、古いメアドを新しいものに更新してやる必要があります。

なお、現状では Native SDK 以外では Email 受け取る術がないので、このままだと Web で Sign in with Apple で登録されてしまうと、登録時にはメアドが取れないことになります。Twitter ログインみたいですね。

## Client Secret の発行ルール

Client Secret というのは通常固定値で、環境変数なりに入れておいてずっと同じ値を使い続けていると思いますが、Sign in with Apple では Client Secret として JWT を生成する必要があります。

詳しい Client Secret の生成方法は [僕の RubyGem の当該部分](https://github.com/nov/apple_id/blob/05b06750b219080022191b6497b0bc768b2c0dd9/lib/apple_id/client.rb#L22) 見ていただくとお分かりかと思いますが、Issuer が Team ID で、Subject に Client ID を指定します。

なお、この JWT には Private Key 毎に Apple が割り振る Key ID を kid ヘッダーに指定することになっているのですが、どうもこの kid を指定しなくても動くようで、kid 指定してない記事がちらほら見受けられます。

Key Rotation 途中の複数鍵が存在するシチュエーションでだけエラーになるとかあるあるなんで、kid は指定するようにしましょうね。

## ID Token の発行ルール

Native SDK や Apple が提供する JS SDK を使うと、フロントエンドで Authorization Code と同時に ID Token が受け取れます。

これには response_type=code+id_token を利用しています。

ただ、通常 OIDC ではフロントチャネルで発行された ID Token には nonce を含める必要があるのですが、Sign in with Apple では nonce 自体サポートされていません。

Replay 攻撃し放題ですね。

もし Replay 攻撃されて困るようであれば、フロントチャネルで受け取った ID Token は華麗にガン無視し、Authorization Code を使って Token Request を送り、バックチャネルで受け取った ID Token だけを利用するようにしましょう。

なおこの場合でも nonce がないのでいわゆる "Code Injection 攻撃" に対しては対処しようがありません。

現状 OAuth Client 側ができる対応としては、Authorization Code を漏らさないように頑張るくらいしかなさそうです。

てか、フロントチャネルで発行される ID Token って、使い道としては Detached Signature くらいしかないはずなんですけど、c_hash もなく nonce もない ID Token 発行する Apple さんは何考えているんでしょうか？

何の使い道もない ID Token を発行しないでください。バカが使って脆弱性を作ってしまうではないですか。

## UserInfo の取得方法

既に述べましたが、Email (と Display Name) は Native SDK でしか取得できません。

Web で登録された時にメアド取れなくて困るってのもあるんですが、そもそもフロントチャネルで取得した UserInfo って、結局バックエンドに送るじゃないですか？

で、バックエンドへの通信って、Charles Proxy みたいないわゆる SSL Proxy 使うと、端末所有者からすれば割と簡単に書き換え可能じゃないですか？

でも、そのメアドは Apple さん的には Verify 済だとおっしゃるんですよ。だから疎通確認メールを各 Client が送信する必要はない、と。

いやいや、フロントチャネルでそんなメアド渡されたって、Verified な状態でそのままバックエンドに送るの大変ですから。

Native SDK では User ID までフロントチャネルで取れちゃうんで、アプリからバックエンドに Email と User ID 送りつけて、それ改ざんしたら他人のアカウントにログインできちゃう〜、みたないアプリが出てきそうなにおいがします。

## User 認証の UX

Safari で Apple 公式 JS SDK を使った場合と Native SDK を使った場合は、OS ログイン時のパスワードとか FaceID / TouchID で認証することでユーザー認証が行えます。

これは UX 素晴らしいです。

UX 的にはほぼほぼ WebAuthn と同じであり、iOS Web では WebAuthn じゃなくて Sign in with Apple でいいんじゃね？みたいな気がしなくもないです。

が、それ以外の環境でのログイン UX は結構辛いものがあります。

Apple ID ってなんかやたら二段階認証設定させようとしてくるし、ついつい二段階認証有効にしてしまうじゃないですか？で、6桁 PIN が iPhone なり Mac に飛んできますよね？

あれを Android ユーザーが Web で使うことを考えてください。

その6桁 PIN が飛んでくる端末、きっとそん時そのユーザーの手元には...ないよねぇ〜。

まぁ SMS で PIN 送るオプションもあるんで、Android 単体で完結しなくもないですけど、ちょっと iOS から Android に乗り換えたユーザーのこと考えると、悪夢です。

## まだまだ続くかな？

もうそろそろ疲れたので今日はこれくらいで。

もうちょい気になる点あるんだけど、それは (2) で書くかもですし、(2) は無いかもです。

あ、ヤフースコア取得用 Client ID が取得できたら、絶対 (2) も書いちゃうな〜。

<blockquote class="twitter-tweet" data-lang="en"><p lang="ja" dir="ltr">Y!J 社に大人な感じで「ヤフースコア確認サービス」という各ユーザーが自分のヤフースコアを確認できるサービスを作りたいから Client IDくれよって依頼を投げてみた。</p>&mdash; nov matake (@nov) <a href="https://twitter.com/nov/status/1135853530072305664?ref_src=twsrc%5Etfw">June 4, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="ja" dir="ltr">Y!J 社に大人な感じで「ヤフースコア確認サービス」という各ユーザーが自分のヤフースコアを確認できるサービスを作りたいから Client IDくれよって依頼を投げてみた。</p>&mdash; nov matake (@nov) <a href="https://twitter.com/nov/status/1135853530072305664?ref_src=twsrc%5Etfw">June 4, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="en"><p lang="ja" dir="ltr">Y!J 社に大人な感じで「ヤフースコア確認サービス」という各ユーザーが自分のヤフースコアを確認できるサービスを作りたいから Client IDくれよって依頼を投げてみた。</p>&mdash; nov matake (@nov) <a href="https://twitter.com/nov/status/1135853530072305664?ref_src=twsrc%5Etfw">June 4, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

自分で OAuth Client 自作できるみなさんにはぜひ Yahoo! JAPAN にヤフースコア取得可能な Client ID 発行を依頼することをお勧めしますし、そうでないみなさんにはヤフースコアのオプトアウトをお勧めします。

[ユーザーの行動や消費を“格付け”「Yahoo!スコア」の作成・利用を拒否する方法](https://news.yahoo.co.jp/byline/fujisiro/20190604-00128752/)

それではいい週末を。