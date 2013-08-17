---
layout: post
title: JSON Web Token (JWT)
tags:
- JustMigrate
---
<p>@novです。</p>
<p>個人的に最近OAuth 2.0よりJWT (というかJWS) を利用するシーンが多く、毎回同じ説明するのもめんどくさいのでブログにまとめるかと思い、どうせならOAuth.jpに書くかということで、こんな記事を書いております。</p>
<p>（そろそろJWTとJWSは、OpenID Foundation Japanの翻訳WGで翻訳するべき？）</p>
<h3>JSON Web Token (JWT) とは、JSONをトークン化する仕組み。</h3>
<p>元々はJSONデータにSignatureをつけたりEncryptionする仕組みとして考えられたものの、Signature部分がJSON Web Signatue (JWS)、Encryption部分がJSON Web Encryption (JWE) という仕様に分割された。</p>
<p>それぞれ2012年10月26日現在の最新仕様はこちら。</p>
<p>（JWTとJWSは既にだいぶ仕様が固まってきているものの、まだIETFのInternet-Draftなので注意）</p>
<ul><li><a href="http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-04">JSON Web Token (JWT, draft 04)</a></li>
<li><a href="http://tools.ietf.org/html/draft-ietf-jose-json-web-signature-06">JSON Web Signature (JWS, draft 06)</a></li>
<li><a href="http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption-06">JSON Web Encryption (JWE, draft 06)</a></li>
</ul><p>JWEはまだ実際に使うケースが無いので、ここでは説明しない。</p>
<h3>Signed JWT</h3>
<p>JWS使ってSignatureつけられたJWT。</p>
<p>HeaderとPayloadとSignatureという3つのセグメントから構成される。</p>
<p>Headerは署名アルゴリズムなどを含むJSONを、URL-safe Base64 Encodingした文字列。</p>
<p>Payloadは実際に送信したいJSONデータそのものを、URL-safe Base64 Encodingした文字列。</p>
<p>Signatureは、HeaderとPayloadを &#8220;.&#8221; で連結した文字列に対して、Headerに指定されたアルゴリズムで署名をして、その署名をURL-safe Base64 Encodingした文字列。</p>
<p>サンプルは<a href="http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-04#section-3.1">JWT Spec Section 3.1</a> (draft 04の場合) を読むこと。</p>
<h3>Signature Algorithms</h3>
<p>サポートされているアルゴリズムは、これまた別仕様のJSON Web Algorithm (JWA) で規定されている。</p>
<ul><li><a href="http://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms-06">JSON Web Algorithm (JWA, draft 06)</a></li>
</ul><p>JWA Section 3 (draft 06 現在) では、JWSの署名アルゴリズムとして以下のアルゴリズムがサポートされている。</p>
<ul><li>HMAC-SHA256</li>
<li>HMAC-SHA384</li>
<li>HMAC-SHA512</li>
<li>RSA-SHA256</li>
<li>RSA-SHA384</li>
<li>RSA-SHA512</li>
<li>ECDSA-SHA256</li>
<li>ECDSA-SHA384</li>
<li>ECDSA-SHA512</li>
</ul><p>正直ECDSA (楕円曲線暗号) ってのは僕もよく理解していなし、Ruby以外で使いたい場合にどう書けばいいかとかさっぱりなので、個人的には共通鍵使う場合はHMAC、公開鍵使う場合はRSAを使っている。</p>
<p>例えば、こんな感じ。</p>
<ol><li>認証サーバーからiOSアプリに渡すデータには (認証サーバーの秘密鍵を使って) RSA-SHA256使って署名</li>
<li>iOSアプリからリソースサーバー (認証サーバーとは別) に渡すJSONには、Step 1で認証サーバーに署名されたJWSに入ってる共通鍵を利用してHMAC-SHA256使って署名</li>
<li>リソースサーバーはiOSアプリからStep 1で発行されたJWSとStep 2で発行されたJWSを同時に受け取って、Step 1のJWSを検証した後そこに含まれてる共通鍵でStep 2のJWSを検証</li>
<li>リソースサーバーはレスポンスに (リソースサーバーの秘密鍵を使って) RSA-SHA256使って署名</li>
</ol><p>（iOSではSHA256よりSHA1の方が使いやすいらしく、HMAC-SHA1とRSA-SHA1使ってたりすることもあるが..）</p>
<h3>各言語のライブラリ (情報求む！)</h3>
<ul><li><span>Ruby </span><a href="https://github.com/nov/json-jwt">json-jwt</a></li>
<li><span>Python </span><a href="https://github.com/rohe/pyjwkest">pyjwkest</a></li>
<li><span>Java </span><a href="https://bitbucket.org/nimbusds/nimbus-jose-jwt/wiki/Home">Nimbus JOSE+JWT</a></li>
<li><span>PHP </span><a href="https://github.com/ritou/php-Akita_JOSE">PHP Akita JOSE</a><span>,</span><span> </span><a href="https://github.com/nov/jose-php">JOSE</a></li>
<li><span>Perl </span><a href="https://github.com/xaicron/p5-JSON-WebToken">JSON::WebToken</a></li>
</ul><p>Rubyのjson-jwtとPHPのJOSE (2番目の方) は僕が作ってるので、README (希望としてはSpecも) とか読んでも使い方分からない場合は僕に直接聞いていただければと思います。</p>
<p>それ以外はあんまり詳しく使い方知らないので、使い方についてはドキュメント (あれば) 読むなりそれぞれの作者に直接聞いてください。それぞれの作者に紹介するくらいならできます。</p>
<p>node.jsとかObjective-CにもJSON Web Tokenのライブラリは見かけるのですが、HMACしかサポートしてなかったりしていまいち使えそうなの見つけられてません。（多分Google Wallet APIでHMACなJWSが使われてるので、それだけサポートしたライブラリがあるんだと予想）</p>
<p>ここにない言語でRSAもサポートしてるライブラリご存知でしたらお教えいただけるとうれしいです。</p>
