---
layout: post
title: "Rubygem for Sign in with Apple & Rails Sample App"
date: 2019-06-04 22:38
comments: true
categories:
---

iOS App に Apple ID でログインできたらいいのになぁ〜と思い続けてはや数年、ついにそんな時代がやってきましたね！

ということで、FB Connect 登場時の fb_graph gem リリース以来のスピード感で、Sign in with Apple 用の ruby gem をリリースしてみました。

[github.com/nov/apple_id](https://github.com/nov/apple_id)

ついでに Rails のサンプルアプリも。

[github.com/nov/signin-with-apple](https://github.com/nov/signin-with-apple)

このサンプルアプリはこちらで動かしてるので、試したい方はどうぞ。

[signin-with-apple.herokuapp.com](https://signin-with-apple.herokuapp.com/)

Sign in with Apple は OpenID Connect を採用してるんですが、現状では Native SDK でしか email や fullName は取れないようです。

そのうち UserInfo API が出てくると思うんで、出てきたら apple_id gem でもサポートしようかと思います。

ps.  
とりあえず Terminal で動かすだけでいいよって方はこちらのサンプルをどうぞ。

https://gist.github.com/nov/993a303aa6badd8447f7b96fb952088e
