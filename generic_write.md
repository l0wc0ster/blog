[back](/blog)

### Don't know the user's password? Make it kerberoastable =)

When the domain user has "GenericWrite" object control rights related to another one, it grants you the ability to write to any non-protected attribute on the target object, for example, you can assign a "$true flag" for the "DoesNotRequirePreAuth" attribute, which will give you the ability to perform an "AS-REP Roasting" attack , or you can assign an "SPN attribute" to the domain user and then you can perform a Kerberoasting attack

But if you are really lucky, then you will get the plain text password from received hashes, hashcat will help u )

![Image](/img/generic_write/1.png)
![Image](/img/generic_write/3.png)
![Image](/img/generic_write/4.png)
![Image](/img/generic_write/6.png)
![Image](/img/generic_write/2.png)
![Image](/img/generic_write/5.png)

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

[back](/blog)