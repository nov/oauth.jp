---
layout: post
title: "Rails SessionにCookieStore使った時の問題点"
date: 2013-09-26 12:45
comments: true
categories:
---

今日 @mad_p さんからRT来てたこのツイートに関して、ちょっと調べたのでまとめときます。

<blockquote class="twitter-tweet"><p>Security Issue in Ruby on Rails Could Expose Cookies <a href="http://t.co/JlsXVEn4rZ">http://t.co/JlsXVEn4rZ</a></p>&mdash; Ruby on Rails News (@RubyonRailsNews) <a href="https://twitter.com/RubyonRailsNews/statuses/383002160654336000">September 25, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### 前提条件

Railsではデフォルトでsessionをcookieにのみ保存して、DBなりmemcacheなりのserver-side storageには何も保存しません。
これがCookieStoreとか呼ばれてるやつです。

この場合のsession cookieは、Railsのsession object (Hash object) をMarshal.dumpしてそれに署名を付けたtokenです。
rails 4では署名付ける代わりに暗号化してるけど、ここで述べる問題に関してはsignedなのかencryptedなのかは関係ないのでその点は無視していいです。

「current user identifierを含むSigned JSONをcookieに入れてるのと同じなので、OpenID ConnectのID Tokenがcookieに入ってるようなもんだと思えばいい」と言えば、#idcon に来たり定期的にOAuth.jp読んだりしてる人には通じるだろうと期待します。

### 問題点

server-sideではstate管理しないので、当然remoteでセッションの無効化はできません。
つまりログアウトしてもsession cookieのtoken自体は無効化されません。

これを確認するには、サイトにログインしてCookie Headerから\_foo_cookieってやつ見つけ出してその値をコピーして、CharlesでRequest Cookie Headerの\_foo_sessionの値を常にそのコピーした値に書き換えてやるようなRewriteルールを設定してやって、サイトからログアウトしてみるとよいです。
通常server-sideでsession管理してるサービスだとこれでログアウトされるけど、CookieStoreを使ってる場合はこの条件の元ではログアウトしてもログイン状態のままになります。

でもまぁこれはCookieStoreの仕組み上どうしようもないのです。

問題は、session cookieが無効化できないのに、それが永遠に有効であること。
FBとかY!Jとかでやってる「パスワード変更したら既存のsessionが無効になる」ってのはRailsは一切面倒見てくれないので、パスワード変えても漏洩したsession cookieは有効なままです。
なので、ひとたびsession cookieが漏れたら、完全にアウト。
なにやってもアカウント乗っ取られたまんま。永遠に。

<!-- more -->

### 解決策

<a href="http://www.bryanrite.com/ruby-on-rails-cookiestore-security-concerns-lifetime-pass/">こちらのブログ記事</a>は2011年のものなので、この問題自体は古くから知られている問題なのだと思います。僕知らなかったけど :p

この記事では以下のように「SSL使えばいいよ」って解決策示されてるけど、SSL化はsession cookieが漏洩する可能性を低くするだけで、「session cookie tokenが永遠に有効である」という状況を回避するものではないです。

<blockquote>
The best and easiest solution is simply to use SSL.  Not just on your login forms and actions, but your entire site, or at least any pages where you have sessions turned on.  With SSL on, the user will not be able to replay your cookies and the entire attack vector is shut down.  Rails 3.1 has a handy force_ssl switch you can use, and you can use something like:
</blockquote>

「session cookie tokenが永遠に有効である」という状況を回避したい場合は、各デベロッパーが <code>session[:expires_at] = 1.hour.from_now</code> とかして session cookie token 自体に有効期限を含めて、それを毎リクエストごとにチェックする必要があります。

OpenID ConnectのID Tokenでも同じ問題が発生しうるので、ID Tokenは有効期限 (exp) をtoken自体に含んでいます。
で、ID Tokenを受け取ったサービスは、署名検証と同時に有効期限もチェックすることで、ID Tokenが漏洩してもなりすましリスクを一定期間内に狭めることができます。
ID Tokenはnonceを含むこともできる (し多分通常はnonce付いてる) ので、onetime tokenとして利用することもできます。
onetime tokenとして扱うと、なりすましリスクはかなり限定的になるでしょう。(ゼロではない)

...ってことで、RailsのsessionでもOpenID ConnectのID Tokenが含んでるような情報を含むようにして、ID Tokenのverificationと同じような処理をsession受け取った際にやってやれば、それで良いのです。
あとはConnectの仕様さえ読めばOKェ

もしくはCookieStoreの代わりにMemcacheStore使うようにしてもいいです。
sessionをserver-sideで管理するようにさえすれば、この問題はそもそも発生しないですし。
パフォーマンスに影響しますけど。

### 捕捉

Railsには<code>config/initializers/session_store.rb</code>には<code>expires_after</code>ってオプションもあるけど、こちらはあくまでcookieの有効期限であって、悪意の無い通常のユーザーが普通にブラウザでアクセスしてればその期間すぎたらブラウザがcookieを削除してくれるけど、悪意あるユーザーはわざとcookieの有効期限無視してsession cookie送りつけてきたりするので、そこの設定はこのケースでは意味ない。

ログアウト処理で<code>session.delete</code> (<code>session.clear</code>だっけ？ちょっと記憶曖昧) すると、RailsはSetCookieヘッダーを返してブラウザからそのsession cookieを削除しようとします。
が、そのsession cookieを削除するかどうかの決定権は、server-sideではなくclient-sideに存在します。