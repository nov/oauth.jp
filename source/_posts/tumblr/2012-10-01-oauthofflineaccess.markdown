---
layout: post
title: OAuthのoffline_accessについて
tags:
- JustMigrate
---
<p>少し前ですが、OpenID Connectのbitbuckeetで<a href="https://bitbucket.org/openid/connect/issue/539/messages-0-add-scope-for-offline-access">offline_accessについての議論</a>がされていたので、まとめておきます。</p>
<h3>Googleの場合</h3>
<h4>offline access の定義</h4>
<ul><li>refresh tokenが発行されるのがoffline。</li>
<li>特にoffline accessを要求しない限り、デフォルトではonline accessを要求したものと見なされる。</li>
<li>online accessの場合はrefresh tokenは発行されない。</li>
<li>refresh tokenはユーザーがrevokeするまで有効。</li>
</ul><h4>auto approval</h4>
<ul><li>一度ApproveされたClientに対しては、accessがrevokeされるまでは次回以降のAuthorization Request時の同意画面はスキップされる。</li>
<li>同意画面がスキップされる場合、refresh tokenは発行されない。</li>
</ul><h4>promptとaccess_type</h4>
<ul><li>Authorization Request時にprompt=trueというパラメーターを指定すると、auto approvalを無効化し、ユーザーに同意画面を見せることができる。</li>
<li>同様にaccess_type=offlineを指定すると、offline accessを要求できる。</li>
<li>access_type=offlineを指定した場合、初回同意時にのみrefresh tokenが発行される。</li>
<li>それ以降もrefresh tokenが必要な場合は、prompt=trueとaccess_type=offlineを同時に指定すること。</li>
</ul><h3>AOLの場合</h3>
<h4>offline accessの定義</h4>
<ul><li>デフォルトではcode flowでは常にrefresh tokenが返される。</li>
<li>refresh tokenはユーザーがログアウトするまで有効。</li>
<li>ユーザーがログアウトしても有効なrefresh tokenが欲しい場合は、scopeにoffline_accessを指定する。</li>
</ul><h4>auto approval</h4>
<ul><li>ユーザーが明示的にチェックボックスにチェックを入れた場合のみ、次回以降の同意画面がスキップされる。</li>
<li>同意画面がスキップされるケースでも、offline_accessを要求することができる。</li>
</ul><p>ところで<a href="https://bitbucket.org/openid/connect/issue/539/messages-0-add-scope-for-offline-access">このIssue</a>、まだopenですね。</p>
