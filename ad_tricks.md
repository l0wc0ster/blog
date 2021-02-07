[back](/)

### From Windows Defender Command Line Utility to Domain Admin

Anyone who has been hacking an Active Directory domain knows that in the beginning it is necessary to pay special attention to the enumeration of open services. For example, it can be very efficient to enumerate domain users by using Kerberos service

![Image](/img/ad_tricks/1.png)

Using standard Windows firewall protection tools, administrators restrict access to services via ipv4, to find out what other interfaces are available for the host, only access to the msrpc 135/tcp service is required. In this case, using the IOXIDResolver interface and SimplePing () and ServerAlive2 () requests, you can get a list of internal interfaces, as well as the server hostname

![Image](/img/ad_tricks/winrm.png)

It is very common for administrators to set up network-level protection for Brute Force attacks on SMB, LDAP, WINRM, HTTP services, and so on. However, it is often forgotten to limit the number of requests to Kerberos service. To brute force passwords for domain users you can generate a list of NTLM hashes and use the getTGT.py utility from the impacket set. In this case, the brute-force attack will target a TGT ticket request with an encrypted timestamp using Kerberos Pre-Authentication. 
This short script will help you iterate through the TGT:

```
for i in $(cat /tmp/hash); do getTGT.py domain.local/domain_user@dc.domain.local -hashes: $i -dc-ip dc.domain.local; echo $i; done | tee /tmp/tgt_log.txt
```

![Image](/img/ad_tricks/2.png)

![Image](/img/ad_tricks/3.png)

![Image](/img/ad_tricks/4.png)

However, do not forget about the password policy in the domain, if it is installed then the brute-force method will block the target account. To query the **Account Lockout Policy**, you can run **network accounts /domain** command. After successfully selecting the password, you will be able to use it to remotely log in with WINRM, SMB, LDAP services, or to load the registry tree using for example:

```
reg.py -k apt.htb.local query -keyName HKU -s command
```

![Image](/img/ad_tricks/5.png)

![Image](/img/ad_tricks/6.png)

The machine **user$** in the domain has the **Replicate Directory Changes** permissions by default, which means you can perform DCSync attacks. Using the **Windows Defender command line utility**, you can perform a reverse authentication request and get the netNTLM hash on the attacker, and then restore the original NTLM using rainbow tables on the crack.sh online service (if you get netNTLMv1)

![Image](/img/ad_tricks/7.png)

![Image](/img/ad_tricks/8.png)

![Image](/img/ad_tricks/9.png)

![Image](/img/ad_tricks/10.png)

To get the NTLM hash of the domain admin, you can use the secretsdump.py utility, from the impacket set by performing a DCSync attack

![Image](/img/ad_tricks/11.png)

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