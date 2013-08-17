---
layout: post
title: 'Re: OAuth 2.0のclient_secretって本当に秘密鍵ですか？'
tags:
- JustMigrate
---
<p>昨日こんな記事を見かけたので、記事にまとめることにします。</p>
<p><a href="http://koduki.hatenablog.com/entry/2012/07/16/113204">OAuth2.0のclient_secretって本当に秘密鍵ですか？</a></p>
<p>元記事にあるとおり、現状Native AppでのOAuth 2.0の実装は、API提供者・利用者ともにポリシーがバラバラで、混乱の元になっていると思います。</p>
<blockquote class="posterous_medium_quote">
<p>Googleのドキュメントにも「the client_secret is obviously not treated as a secret.」とあるわけだけど、そのくせclient_secretを使ってるし、ネットで調べても少なくない数の人がアプリに埋め込んでるので、client_secretを公開したときの問題を考えてみる。</p>
</blockquote>
<h3>"offline" アクセスと "online" アクセス</h3>
<p>Googleは、<strong>"offline access"</strong> に対して以下のようなポリシーを持っています。</p>
<p><a href="http://googleappsdeveloper.blogspot.jp/2011/10/upcoming-changes-to-oauth-20-endpoint.html">Upcoming changes to OAuth 2.0 endpoint - Google Apps Developer Blog</a></p>
<p><a href="https://bitbucket.org/openid/connect/issue/539/messages-0-add-scope-for-offline-access">openid / connect / issues / #539 - Messages - 0. Add scope for offline access - Bitbucket</a></p>
<p>上の記事では議論が長くてかつ英語なので、簡単に要約すると、</p>
<ul><li>access tokenが無効化されると、clientは新しいaccess tokenを取得しなければならない。</li>
<li>新しいaccess tokenを受け取るには、ユーザーがその場にいる必要がある。<strong>ユーザーが既にclientのアクセスに同意している場合は、同意画面をスキップさせることができる</strong>ので、ユーザーが毎回Googleの同意画面を見る必要は無い。</li>
<li>JS Appなど、ブラウザ内で動作していて常にユーザーとインタラクションしているclientは、同意画面さえスキップできればリダイレクトベースで必要に応じて毎回access tokenを取得すればよい。</li>
<li>スマホ上のNative Appの場合は、UX的に毎回ブラウザ経由してaccess tokenを取得するのはつらいので、<strong>ユーザーから明示的に &#8220;offline&#8221; アクセスへの同意を得て</strong>、次回以降はrefresh tokenを使って新しいaccess tokenを取得すればよい。</li>
</ul><p>そして、Googleは、refresh tokenを伴う場合を &#8220;offline&#8221; アクセス、伴わない場合を &#8220;online&#8221; と定義しています。詳細は異なるものの、似たような定義はFacebookやAOLなども行っています。</p>
<h3>Native Appに埋め込まれたclient_secretは簡単に漏洩する</h3>
<p>このご時世 <a href="http://www.charlesproxy.com">Charles</a> などを使えば、自分のデバイス上で行われているSSLリクエストなら簡単に覗き見ることができます。</p>
<p>OAuth 2.0では、authorization codeとaccess tokenを交換する時とrefresh tokenをaccess tokenと交換する時に、client_secretを平文でAuthorization Server (この場合はGoogle) に送信します。</p>
<p>そのため、上記2つのいずれかのリクエストがiPhone上のNative Appから発行されるなら、Jailbreakや逆コンパイルなどせずともエンドユーザーなら誰でも、そのアプリに埋め込まれたclient_secretを知ることができます。</p>
<p>OAuth 2.0のclient_secretをアプリに埋め込んで配布するというのは、client_secretが漏洩する前提でそれを扱っているということです。そしてその場合、当然以下のような疑問がでてきます。</p>

<h3>client_secretが漏洩してはいけないのか？</h3>
<p>これに関しては、例えばFacebook Graph APIの場合であれば、client_secretが漏洩すると以下のようなリスクがあります。(個人的にGoogle APIは使ってないので、Googleに関しては僕はよく把握していません)</p>
<blockquote class="twitter-tweet">
<p><a href="https://twitter.com/gabunomi_">@<strong>gabunomi_</strong></a> FB Graph APIなら、client_secretさえあれば、任意のユーザーをそのアプリからBanしたり、Banを解除できたりするはずです。まぁ4sqユーザーを全員Banすれば、FBがそれを検知してなんらか対応してくるとは思いますが。</p>
— nov matakeさん (@nov) <a href="https://twitter.com/nov/status/230990719257559040">8月 2, 2012</a></blockquote>

<p>client_secretが漏洩した場合のリスクはAPI提供者側のポリシーに依存するため、Generalな解答をするとすれば以下のようになるでしょう。(ちなみに、いまClient Credentials Flowがサポートされていないからといって、将来にわたってその状態が維持される保証があるわけではないです。その場合はclient_secretを発行しなおしたりするんでしょうかね？ &gt; だれとなく)</p>
<blockquote class="twitter-tweet">
<p><a href="https://twitter.com/gabunomi_">@<strong>gabunomi_</strong></a> API提供者側に「なんでここでclient_secret埋め込ませてんの？」って問い合わせて、明確な回答が返ってくるなら、1 clientとしてはそれでOKかと。「いろいろあってしょうがなく」とか言われたら危険です。被害受けるとしたらユーザーじゃないあなた。</p>
— nov matakeさん (@nov) <a href="https://twitter.com/nov/status/230997139705188352">8月 2, 2012</a></blockquote>

<p>全てのAPI提供者がNative Appに埋め込まれたclient_secretが漏洩することを前提としたアクセスコントロールを実施しているわけでも無いようですし、そもそもNative Appにclient_secretを埋め込むことを禁止しているFacebookのようなOAuth Serverも存在する (=&gt; 本来こっちの方が一般的であるべき) ので、Developerにとっては混乱の元でしょうね。</p>
<p>Native Appにclient_secretを埋め込む前提で、その漏洩の可能性という点で見れば、OAuth 1.0を使った方が良い場合もあるでしょう。OAuth 1.0では、Native Appにclient_secretを埋め込んでもそれは署名計算に使われるだけで直接OAuth Serverに送信されることはありません。かといっていまからOAuth 1.0を採用するというのは、時代の流れ的にどうなんだということもありますが。</p>
<blockquote class="twitter-tweet">
<p><a href="https://twitter.com/gabunomi_">@<strong>gabunomi_</strong></a> OAuth 2.0では（JWT assertionなどを使わない限り）token endpointにclient secretを平文で送るので、埋め込まれたsecretは逆コンパイル無しでも容易にsecretは漏洩します。そこはOAuth 1.0のがまし。</p>
— nov matakeさん (@nov) <a href="https://twitter.com/nov/status/231186384424157184">8月 3, 2012</a></blockquote>
<h3>GoogleとFacebookのスタンスの違い</h3>
<p>GoogleがNative App (= Installed Application) 用のフローを用意してNative Appにclient_secretを埋め込むように案内する一方、Native AppにはFB Official Appと連携したフローを用意してclient_secretを不要にしているFacebookの存在は、対照的です。</p>
<p>Googleの場合は、Client登録時にClientがWeb AppなのかNative Appなのかといった分類をさせているはずで、そのclient typeによってclient_secretを使ってできることに差を付けているのかもしれません。(<strong>要確認</strong>: Google APIよく知らないので、この辺詳しい人いたら教えてほしい)</p>
<p>Facebookの場合は、Native AppとWeb App、Facebook iFrame Appといった複数の種類のClientを、提供者が同一であれば同じclient_id (&amp; client_secret) を使い回せるようにして、cross-platformな環境でアプリを提供しやすくしているため、client typeによってclient_secretの価値を変えるということはやりづらいのかも知れません。(じゃあGoogleはcross-platformなClientに対してどう考えてるの？ってのは、Googlerに直接聞いてみたいかも)</p>
<h3>client_secretが漏洩した時に被害を受けるのは誰？</h3>
<p>これもまた各OAuth ServerがどんなAPIを提供していて、OAuth 2.0のClient Credentials Flowを使って得たaccess tokenで何ができるのかに依存するので、一概には言えないのですが、よくあるケースとしてはAPI利用状況のAnalytics情報を取得したりするAPIが考えられるので、被害を受けるのはエンドユーザーというよりはClient Developer自身であることの方が多いでしょう。</p>
<p>client_secretを埋め込む実装をしているOAuth Client Developerは、一度利用しているAPIがClient Credentials Flowをサポートしているのか、Client Credentials Flowで得たaccess tokenでは何ができるのか、一度APIドキュメントを確認したりAPI提供者に問い合わせてみた方が良いかも知れません。</p>
<p>Facebookの用にclient_secretさえあれば任意のユーザーをBanできてしまったりする場合は、client_secretが漏洩することでClient Developerとエンドユーザー両方が被害を受けることもありえます。</p>
<h3>蛇足: オレはこう思う！（だっけ？）</h3>
<p>まぁこの辺りはOAuth 1.0からOAuth 2.0になってServer / Client双方にいろいろ選択肢が増えたので、各社バラバラな仕様になってしまって</p>
<blockquote>
<p>When compared with OAuth 1.0, the 2.0 specification is more complex, less interoperable, less useful, more incomplete, and most importantly, less secure.</p>
<p><a href="http://hueniverse.com/2012/07/oauth-2-0-and-the-road-to-hell/">OAuth 2.0 and the Road to Hell</a></p>
</blockquote>
<p>っていう前OAuth 2.0 Authorの彼の意見も、あながち無視できないところではある。</p>
