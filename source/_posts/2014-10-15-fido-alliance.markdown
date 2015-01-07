---
layout: post
title: "FIDO Alliance"
date: 2014-10-15 11:28
comments: true
categories:
---

先週末、FIDO Alliance　のメンバーが来日して、[FIDO Alliance Tokyo Seminar](http://www.dds.co.jp/fido_tokyoseminar/) というセミナーが開催されていました。僕も今春の [#idcon vol.18](http://idcon.doorkeeper.jp/events/10195) で [FIDO Alliance について紹介した](http://www.slideshare.net/matake/fido-alliance) 時にまだいろいろ不明点が多かったのもあって、このセミナーに参加してきました。

で、参加してみて分かった事。

## 生体認証押し

まぁ、これは FIDO Alliance のサイト見ても、元々そうっちゃそうなんですが。

FIDO は仕様的には特に生体認証センサーとか必要ないし、FIDO に準拠してない Apple TouchID みたいなセンサーモジュールと連携したソフトウェアモジュールさえ積んでればFIDO 仕様に沿った実装は可能です。

ひとことで言うと、エンドユーザーが「端末のセキュア領域に保存されている秘密鍵にアクセスして assertion に署名できる存在」であることを認証しているに過ぎないので、その秘密鍵のロック解除に指紋使ってもいいし、PIN Code 使ってもいいはずです。

が、やはり Alliance Member には生体認証ハードウェアモジュールのベンダーが多い事もあってか、発表ではヤケに生体認証を押していました。

その影響か、Twitter とか現地での Q&A でも、生体認証のセキュリティについての質問が多かったように思います。

<!-- more -->

## UAF と U2F は完全に別物

FIDO Alliance では、パスワードレスな認証の実現を目指す Universal Authentication Framework (UAF) と、2要素認証の UX 改善を目指す Universal Second Factor (U2F) という2つの Working Group が、それぞれに仕様策定を進めています。

僕の #idcon vol.18 時点の理解では、U2F は UAF とほぼ同じで、JS API や Android SDK などでの利用を想定した仕様策定がそれにプラスαで進んでいるのだと思っていたのですが、懇親会で Google の Dirk Balfanz さんに聞いたところでは、U2F は「JS API (と Android SDK)」のみの標準化を行っているそうです。

UAF では、センサーモジュールと Client App、Client App と Backend Server などの間でのやりとりが規定されているのですが、U2F ではその辺りのやり取りは「各実装が自由にやれ」という姿勢とのこと。

UAF 側の Rolf Lindemann (Nok Nok Labs) に別途聞いたところでは U2F Working Group に UAF で策定した仕様を取り込むよう働きかけているとのことでしたが、2つの WG の間の連携は、それほど強くないのかもしれません。

## Discovery は貧弱

FIDO UAF Protocol では、FIDO Alliance が FIDO Ready な製品を作る Vendor に Vendor ID を発行し、FIDO Ready な Product を Metadata Service (?) に登録したりするというようなことが書かれているのですが、現状は Vendor ID の発行にとどまっており、Metadata は必要に応じて RP が「ダウンロード」している状況のようです。

この部分がマニュアルだと、結局デバイスメーカーやアプリベンダーは、OAuth のクライアント登録的なマニュアル作業から抜け出せないように思います。

ま、もろもろまだまだ、って感じでしょうか。

