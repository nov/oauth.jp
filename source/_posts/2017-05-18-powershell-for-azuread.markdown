---
layout: post
title: "PowerShell for AzureAD"
date: 2017-05-18 14:17
comments: true
categories:
---

Windows 10 で AzureAD Module インポート済の PowerShell を気軽に起動できるようにする方法メモ。

## 前提条件

前提として、PowerShell に AzureAD アクセスに必要な Module がインストールされているものとします。

Module インストールは以下のコマンドを Administrator として実行するだけです。

```ps1
Install-Module MSOnline, AzureAD
```

[こんな ext](https://msdn.microsoft.com/ja-jp/library/jj151815.aspx#bkmk_installmodule) は別にいらないです。以前はこいつのインストールでいろいろはまったんですが、あれはなんだったんでしょうか...

## ショートカット作成

まずは PowerShell 自体のショートカットを作成します。

![PowerShell for AzureAD](/images/posts/azure/powershell-for-azure-ad.png)

そしておもむろに Target のところを下記の PowerShell Script に変更。

```ps1
PowerShell -noexit "Import-Module MSOnline, AzureAD; Connect-MsolService"
```

そして Advanced Options の "Run as Administrator" にチェック。

これで、このショートカットダブルクリックするだけで、自動でログイン画面まで Popup してくれるようになります。

## ショートカット起動

起動したら自動でこうなります。

![PowerShell for AzureAD Launched](/images/posts/azure/powershell-for-azure-ad-launched.png)

またちょっと Windows 力上がった気がする。