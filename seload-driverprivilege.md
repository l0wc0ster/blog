[back](/blog)

### Privilege Escalation via SeLoadDriverPrivilege

Sometimes a low privilege user has a "SeLoadDriverPrivilege" attribute that allows him to load and unload drivers on the system.

You can easily use this to increase system privileges by loading your driver and then use vulnerable feature in Capcom.sys. After this your code will be executed in the context of the kernel with system privileges.

More information at: [ExploitCapcom](https://github.com/tandasat/ExploitCapcom)

![Image](/img/seload-driverprivilege/1.png)
![Image](/img/seload-driverprivilege/2.png)
![Image](/img/seload-driverprivilege/5.png)
![Image](/img/seload-driverprivilege/4.png)
![Image](/img/seload-driverprivilege/3.png)

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