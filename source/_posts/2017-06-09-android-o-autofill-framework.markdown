---
layout: post
title: "Android O AutoFill Framework"
date: 2017-06-09 12:47
comments: true
categories:
---

どうも、iPhone ユーザーの Nov です。

Android OS の進化は素晴らしいと思います。  
個人的には、普段使いのスマホじゃなければ Android 一択です。

Chrome の進化も素晴らしいと思います。  
個人的には、普段使いの (ry

最近は Android O から登場した [Autofill Framework](https://developer.android.com/preview/features/autofill.html) ってのが気になってます。

Developer Document で *"Apps that use standard views work with the Autofill Framework out of the box"* とか言われてるんで、Android App で普通にログインフォーム作れば、勝手に ID & Password が Autofill されるっぽいですね。

え？ID & Password 以外にも、住所やクレカ番号も Autofill されるって？

**でしょうね。**

## Autofill Framework Sample App 動かしてみよう

ということで、[Android O Beta](https://www.google.com/android/beta) 入れて、[Autofill Framework Sample](https://github.com/googlesamples/android-AutofillFramework) 動かしてみましょう。

[Android Studio 3.0 Preview](https://developer.android.com/studio/preview/index.html) とかいうのもダウンロード必要なようです。

なんかいろいろ足りねぇとかエラー出るけど Java も Kotlin も Android Studio もよぉわからんので、まぁポチポチしていろいろインストールします。

で、Android O Beta をインストールした Nexus 5X (普段使いではない) をターゲットにしてアプリをビルド...

ジャジャーン！！

<!-- more -->

![Autofill Framework Sample Build Error](/images/posts/android/autofill-framework/sample-build-error.png)

...はい、端末側の Android O Beta が古かったです。アップデートします。

普段使いじゃないから電池残量が不足してアップデートするまえにまず充電しろとか言われる。

これだから Android は... #違

で、もろもろ整えて、再チャレンジ！！

ジャジャーン！！

![Autofill Framework Sample Build Error 2](/images/posts/android/autofill-framework/sample-build-error2.png)

この Sample App の最新版リリース以降に、Android 側の API が変わったのか...?

Android Studio 3.0 Canary 2 を入れれば解決しそうな気もするが、Canary 2 が見つからない...

**もういぃ！！その辺に転がってるアプリで試す！！**

## Cookpad で試してみよう

ジャジャーン！！

![Cookpad](/images/posts/android/autofill-framework/cookpad.png)

ここ数年いろいろ大変そうな Cookpad さん (普段使いではない)。

![Cookpad - Login](/images/posts/android/autofill-framework/cookpad-login.png)

ログイン画面...うん、普通に実装されてそう。 #適当

で、ログインしてみると...

![Cookpad - Store Credentials](/images/posts/android/autofill-framework/cookpad-store-credentials.png)

おぉ！なんかパスワード保存しおる！！

もちろん「OK」

さて、ここでおもむろに [passwords.google.com](https://passwords.google.com) にアクセスしてみましょう。

ありましたね、Cookpad。

![passwords.google.com](/images/posts/android/autofill-framework/passwords-google-com.png)

ただ、Cookpad さんは [SmartLock for Password on Android](https://developers.google.com/identity/smartlock-passwords/android/) も使ってるぽいので、そちら経由で passwords.google.com にパスワード保存したんじゃね？って疑惑をもたれるかもしれません。

もひとつ別アプリでもやってみましょう。

## Hotpepper グルメで試してみよう

ジャジャーン！！

![Hotpepper グルメ](/images/posts/android/autofill-framework/hotpepper.png)

Hotpepper グルメ。普段使い。 **#決してステマではない**

![Hotpepper グルメ - Login](/images/posts/android/autofill-framework/hotpepper-login.png)

ログイン画面...うん、普通に実装されてそう。 #適当

で、ログインしてみると...

![Hotpepper グルメ - Store Credentials](/images/posts/android/autofill-framework/hotpepper-store-credentials.png)

おぉ！なんかパスワード保存しおる！！ **#デジャブ**

![Hotpepper グルメ - Login Autofill](/images/posts/android/autofill-framework/hotpepper-login-autofill.png)

ログアウトしなおして再度ログインしてみたら、Autofill 効いてますね。

![passwords.google.com](/images/posts/android/autofill-framework/passwords-google-com2.png)

passwords.google.com にも保存されてる。

うん、Chrome のパスワードマネージャーとほぼ同じ挙動が、Native App 内のログインフォームにもやってきました。

あとは、Hotpepper グルメの Android App に紐づいて保存されたパスワードを、Chrome で [hotpepper.jp](https://www.hotpepper.jp) ドメイン開いた時にも使えて、その逆もまた可能であれば、SmartLock for Password on Android と同じことが、特にアプリ側で特別な実装しなくても実現できますね。

また、[1PasswordによるAndroid OのAutofillフレームワークの動作デモ](http://juggly.cn/archives/223100.html) とかみると、1Password に保存したパスワードを Autofill Framework 経由で各アプリが受け取れるようです。

Android の設定アプリの *System > Languages & input > Advanced > Input assistance > Autofill service* にいくと、いまのところ Google (= passwords.google.com) しか選択肢ないけど、きっとここに 1Password とかが出てくるんでしょう。

1Password ユーザーとしては嬉しい限りですが、ここでも twitter.com ドメインに紐づいたパスワードを Twitter Android App から呼び出せるのか、は気になるところです。

まぁ、SmartLock for Password on Android でできて、Autofill Framework でできない理由なんてどこにもないですし、できるんでしょうけど。

## SmartLock for Passwords on Android 対応アプリで試してみた

ようするに Cookpad ね。

Autofill Framework 経由で保存されたパスワードが、Chrome で cookpad.com 行ってみても補完されるか試してみたんですが...

**うんともすんとも言わなかったです。**

別の Android 端末で別の Google アカウントに SmartLock for Passwords on Android 経由で保存されたパスワードは補完されたんですけどねぇ...

現時点の Android O では、SmartLock for Passwords on Android 対応時にサイトとアプリ紐付けしてあっても、Autofill Framework には反映されないようです。

ちぇっ...

## 補足

ちなみに Google さんは、OpenID Foundation で [OpenYOLO](http://openid.net/wg/ac/) っていうプロジェクトもやってて、OpenYOLO プロトコルに従ったパスワードマネージャーアプリ (1Password etc.) を Android OS 経由で呼び出せるようにしようとしたりもしてますが、これが Autofill Framework とどういう関係性にあるのかは、気になるところです。

Google さん、おんなじようなことできるもの複数実装して、命名規則もなにもあったもんじゃない感じで、ワクワクハラハラです。

最後に [SmartLock for Passwords on Android 担当イケメン](https://developers.google.com/identity/smartlock-passwords/android/) で同じみの [@agektmr](https://twitter.com/agektmr) を晒して、締めさせていただきます。

ありがとうございました。

<iframe width="560" height="315" src="https://www.youtube.com/embed/cY77sSctzec" frameborder="0" allowfullscreen></iframe>
