---
layout: post
title: 個人情報保護法改正案に見る第三者提供記録義務と越境問題
date: 2015-03-30 14:19
comments: true
categories:
---

先日の「[属性単位のトレーサビリティについて](/blog/2015/02/28/traceability-on-each-attributes/)」という記事でも取り上げましたが、[個人情報保護法改正案](http://www.cas.go.jp/jp/houan/150310/siryou3.pdf)では、第二十五条に「第三者提供に係る記録の作成等」という規定が設けられ、第三者提供時に提供者受療者双方に記録義務が発生することとなっています。

また土曜日に行われた情報法制研究会第1回シンポジウムこの提供記録義務と、同じく改正案第二十四条の「外国にある第三者への提供の制限」を組み合わせると、AWSにデータ保存 (委託行為？) するだけで、第三者提供扱いとなり、記録義務が発生してしまうのではという指摘がありました。(まとめからのリンク先、板倉先生の資料参照。パスワードは空気読んで頑張って探してください。)

[20150328情報法制研究会 第1回シンポジウム「改正個人情報保護法の内容と今後の課題」関連まとめ(私家版)](http://togetter.com/li/801181?page=1)

確かに第二十四条では、認定国もしくは認定事業者以外の外国事業者に対するデータ提供の場合は、下記のように第二十三条の例外規定 (委託や共同利用を第三者提供として扱わないという規定含む) を適用しないと明記しているので、委託の場合でも第三者提供扱いとなり、記録義務が発生することになりそうです。

<blockquote>この場合においては、同条の規定は、適用しない。</blockquote>

さらに外国事業者への提供時には、通常の同意とは別に外国事業者への提供であることを明記した上での同意を取得する必要があるため、海外のRPを相手にする国内IdPは、

* RPが国内事業者であるか否か
* RPが海外事業者である場合には認定国の事業者であるか否か
* RPが非認定国の事業者である場合には認定事業者であるか否か

を判断した上で、いずれにも当てはまらない場合は通常とは別の同意文言を提示する必要が出てきそうです。

いやぁ〜、Dynamic Registrationとかしてる場合じゃないっすね！

どうしましょか、これ？w (ノーアイデァなぅ)