---
layout: post
title: "FB Message API Callback as an Azure Function"
date: 2016-04-19 10:56
comments: true
categories:
---

今日は Azure Function で FB Message API Callback を作ってみます。

Azure Function は、Azure Portal の Marketplace で "Function App" って検索すると出てきますね。

![Azure Function in Marketplace](/images/posts/azure/azure-function-in-marketplace.png)

Function App の Deploy がおわったら、QuickStart から "Webhook + API" を選びましょう。

![Azure Function QuickStart](/images/posts/azure/azure-function-quickstart.png)

以下の様な Node.js のテンプレートアプリが出来上がります。

![Azure Function Template](/images/posts/azure/azure-function-template.png)

まずは FB Message の WebHook としてこの Function を登録します。

<!-- more -->

Azure Function の `Function URL` を FB Message API の `WebHook Callback URL` に登録して、適当な `verify_token` を設定します。

![FB Message API Callback (FB WebHook)](/images/posts/azure/fb-message-callback.png)

WebHook Verification のために、テンプレの Azure Function を以下の様に書き換えます。

```js
module.exports = function(context, req) {
  if (req.query['hub.verify_token'] === 'verify-me') {
    context.res = {
      body: req.query['hub.challenge']
    };
  }
  context.done();
};
```

これで FB 側の `Verify and Save` ボタンを押せば、WebHook の Verifycation が成功して WebHook の登録が完了します。

では次に、Text Message を Echo するように Azure Function を書き換えましょう。

まずは FB Graph API の Messaging API を叩くために必要な FB Page Token を、Azure Function の `Application Setting > App Setting` に設定しておきます。

この時に使う FB Page Token は、先ほど WebHook を登録したページ (FB Messenger API の設定ページ) で取得したものを使う様にしてください。そこで取得した FB Page Token は Expire しません。

![Azure Function Env](/images/posts/azure/azure-function-env.png)

あとは Azure Function 側を以下の様に書き換えてやれば OK です。

```js
var https = require('https');

var sendTextMessage = function (sender, text, context) {
  postData = JSON.stringify({
    recipient: sender,
    message: {text: text}
  });
  var req = https.request({
    hostname: 'graph.facebook.com',
    port: 443,
    path: '/v2.6/me/messages',
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer ' + process.env.APPSETTING_FB_PAGE_TOKEN
    }
  });
  req.write(postData);
  req.end();
};

module.exports = function (context, req) {
  messaging_evts = req.body.entry[0].messaging;
  for (i = 0; i < messaging_evts.length; i++) {
    evt = req.body.entry[0].messaging[i];
    sender = evt.sender;
    if (evt.message && evt.message.text, context) {
      sendTextMessage(sender, evt.message.text, context);
    }
  }
  context.done();
};
```

早速該当 FB Page にメッセージを送ってみましょう。Echo が返ってくると思います。

さてと、YAuth.jp の問い合わせ対応 Bot を作ってみるかな。