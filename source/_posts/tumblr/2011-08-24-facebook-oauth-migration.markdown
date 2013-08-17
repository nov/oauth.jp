---
layout: post
title: Facebook の OAuth Migration で大混乱が起こる前に。
tags:
- JustMigrate
---

2011/10/01、Facebook が認証まわりの API 仕様を変更します。結構大きな変更点の為、Developer としても、FB Canvas アプリもしくは Facebook Connect サイトの1ユーザーとしても、いろいろ混乱を経験する可能性もあるでしょう。
この記事では、大混乱が起こる前に、実際の変更点とDeveloper としてできることをまとめます。

### 変更点1

Graph API 以前に使われていた “fb_sig” や “fb_sig_session_key” といったパラメーターを使った認証方式が、一切使えなくなります。これにより、古い認証方式を使っている Facebook Canvas アプリおよび「Facebook ID でログイン」対応サイトで、ユーザーの認証ができなくなります。

これらのアプリは Graph API 以降に登場した OAuth 2.0 ベースの認証方式に切り替える必要があります。Canvas アプリなら Signed Request、Facebook Connect サイトであれば Facebook JS SDK の最新版を使うようにしてください。

### 変更点2

Graph API 以後のOAuth 2.0ベースでの「Facebook ID でログイン」に対応していたサイトでも、JS SDK がセットする Cookie に含まれる Access Token を利用しているサイトでは、Cookie に Access Token が含まれなくなるため、不具合がでる可能性があります。

<fb:login-button> もしくは Facebook JS SDK の FB.login() を使っているサイトはこれに該当するため、サーバーサイドでのライブラリアップデートやコード変更などが必要になるでしょう。

### Developer としてできること

日本国内では Graph API 以前の Facebook API を利用していた方は多くないと思いますが、もしいた場合は、お使いのライブラリが既にメンテナンスされていない可能性が高いです。Ruby でも Facebooker というライブラリが古い Facebook API 用に存在しますが、既に1年以上アップデートがありません。そのようなライブラリを使っている場合は、違うライブラリに移行するべきでしょう。DEADLINE が 2011/10/01 なので、いますぐ対応を開始することをお勧めします。

Graph API 以降の認証方式を使っていた場合は、コードの変更が必要かどうかは実装依存のため、ひとまず Facebook Developer Apps から該当アプリ（もちろんテスト環境用のやつね）を選択し、Advanced タブの Migrations の（たぶん）一番下にある「OAuth Migration」を Enabled にしてみてください（スクリーンショット参考）。すると該当アプリでは 2011/10/01 以降と同様の動作を確認できるので、不具合の有無を調べましょう。不具合があれば、ライブラリのアップデートなどが必要になるでしょう。

### Developer じゃなかったら？

例えば僕は Ustream にログインするのには Facebook アカウントを使っています。Slideshare もそうです。さすがにこれらのサイトは混乱無く OAuth Migration を終えてくれると信じたいですが、万が一のこともありますよね。

そういう場合、もしそのサイトが「Twitter ID でログイン」とか「OpenID でログイン」に対応していれば、Facebook ID 以外でもログインできるようにしておくことをおすすめします。

まぁ古くから Facebook と連携していて、最近アップデートされてなさそうなサービスの場合は … 人生あきらめも重要です。