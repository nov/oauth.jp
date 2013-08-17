---
layout: post
title: WebIntent x OpenID Connect (en)
tags:
---
<iframe src="http://www.slideshare.net/slideshow/embed_code/12296336" width="400" height="334" frameborder="0" marginwidth="0" marginheight="0" scrolling="no"></iframe><br/><p>When you access to an OpenID Relying Party (RP), you&#8217;ll see 5 or more OpenID Provider (OP) logos at its login page. At worst, you can see 10+ OP logos, even though more than half of them are totally unknown for you. It&#8217;s called &#8220;NASCAR Problem&#8221;.</p>

<p>OpenID community had been trying to solve the problem for a long time, but not much progress on it.</p>

<p>Now, it&#8217;s the era of HTML5, and browsers-side functionality is improving very much.
I found <a href="http://webintents.org/">HTML5&#8217;s WebIntents</a> [<a href="http://dvcs.w3.org/hg/web-intents/raw-file/tip/spec/Overview.html">W3C draft spec</a>] as a browser-based &#8220;discovery&#8221; protocol, which can be a solution for OpenID&#8217;s NASCAR problem.</p>

<p>So that I made an OpenID Connect Provider &amp; Relying Party which relying on the discovery part to WebIntents.</p>

<p>You can play my demo following the below steps.</p>

<ol><li>Access to <a href="https://connect-op.heroku.com/">Nov OP</a> which has <intent> tag in its HTML  tag. Your browser will automatically register this site as a service provider of &#8220;OpenID Connect Discovery&#8221;.</intent></li>
<li>Access to <a href="https://connect-rp.heroku.com/">Nov RP</a>.</li>
<li>Click “Or Try WebIntents?” button, which initiate WebIntents-based OpenID Connect Discovery flow.</li>
</ol><p>Then you&#8217;ll see a small popup which let you choose an OP.
After you choose Nov OP, you will go back to Nov OP and see an alert popup which shows raw OpenID Connect discovery result.
Once RP received the response, it does normal OpenID Connect login flow.</p>

<p>One of my friends, Ryo, made his <a href="https://openidconnect.info/">sample OP</a> &#8220;WebIntent-able&#8221;, so once you access to Ryo&#8217;s OP, you can see 2 OPs at the popup window of intent candidates.</p>

<p><br/>
ps.<br/>
For some reason, this demo works only on Safari.
(probably because of webintents.org&#8217;s JS shim issue?)</p>
