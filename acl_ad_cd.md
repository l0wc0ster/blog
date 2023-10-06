[back](/blog)

### Difference in delegation abuse techniques

We often come across the delegation of services or resources in Active Directory. But sometimes it is difficult for us to find the necessary ways of abuse for such privileges. In this connection, I decided to compile a manual that will help with determining operating techniques for different types of delegation.

A few words about searching for insecure ACLs in Active Directory:

I recently ran into a problem collecting a rather important attribute **ActiveDirectoryRights: Self**.
The fact is that when collecting an AD dump, for example through BloodHound Collector, this attribute is not taken when building object links. However, his abuse will help the user add himself to the appropriate group to which the user's SecurityIdentifier is assigned. More details are described [here](https://zflemingg1.gitbook.io/undergrad-tutorials/active-directory-acl-abuse/self).

```
ObjectDN               : CN=ServiceMgmt,CN=Users,DC=domain,DC=ru
ObjectSID              : S-1-5-21-4078382237-1492182817-2568127209-7683
ActiveDirectoryRights  : Self
SecurityIdentifierName : domain\jaki
BinaryLength           : 36
AceQualifier           : AccessAllowed
IsCallback             : False
OpaqueLength           : 0
AccessMask             : 8
SecurityIdentifier     : S-1-5-21-4078382237-1492182817-2568127209-7682
AceType                : AccessAllowed
AceFlags               : None
IsInherited            : False
InheritanceFlags       : None
PropagationFlags       : None
AuditFlags             : None

```

By the way, to display SecurityIdentifierName, my good friend [crypt0p3g](https://github.com/crypt0p3g) made a cool script that allows you to search for ACLs on AD objects and compare them with SecurityIdentifier, without having to find out who owns the SID:

```
ACL AD Scanner

$username = "domain.ru\jaki";
$pass = 's4per$ecu4r3';
$domain = "domain.ru";
$dc = "dc01.domain.ru";

$cred = New-Object System.Management.Automation.PSCredential($username, (ConvertTo-SecureString $pass -AsPlainText -Force));
$DS=Get-DomainSID -Domain domain.ru -Server $dc -Credential $cred;

#to get group ACL and match with object who have rights
Get-DomainGroup -Domain $domain -Server $dc -Credential $cred | Get-DomainObjectAcl -Domain $domain -Server $dc -Credential $cred |? {if ($_.SecurityIdentifier -match $DS+"-\d{4,5}") { add-member -InputObject $_ -MemberType NoteProperty -Name SecurityIdentifierName -Value ($_.SecurityIdentifier | Convert-SidToName -Domain $domain -Credential $cred -Server $dc)[1]; $_}}

#to get object ACL and match with object who have rights
Get-DomainObject -Domain $domain -Server $dc -Credential $cred | Get-DomainObjectAcl -Domain $domain -Server $dc -Credential $cred |? {if ($_.SecurityIdentifier -match $DS+"-\d{4,5}") { add-member -InputObject $_ -MemberType NoteProperty -Name SecurityIdentifierName -Value ($_.SecurityIdentifier | Convert-SidToName -Domain $domain -Credential $cred -Server $dc)[1]; $_}}

```

This can be done differently if you need to look at the ACL for a specific object, but you have to run several commands and look at the SecurityIdentifier with your eyes.

```
Get-DomainObjectAcl SERVICEMGMT -Domain domain.ru -DomainController dc01.domain.ru

ObjectDN              : CN=ServiceMgmt,CN=Users,DC=domain,DC=ru
ObjectSID             : S-1-5-21-4078382237-1492182817-2568127209-7683
ActiveDirectoryRights : Self
BinaryLength          : 36
AceQualifier          : AccessAllowed
IsCallback            : False
OpaqueLength          : 0
AccessMask            : 8
SecurityIdentifier    : S-1-5-21-4078382237-1492182817-2568127209-7682
AceType               : AccessAllowed
AceFlags              : None
IsInherited           : False
InheritanceFlags      : None
PropagationFlags      : None
AuditFlags            : None

Get-DomainObject  S-1-5-21-4078382237-1492182817-2568127209-7682 -Domain domain.ru -DomainController dc01.domain.ru

logoncount                    : 74
badpasswordtime               : 4/9/2023 12:54:33 PM
distinguishedname             : CN=jaki,CN=Users,DC=domain,DC=ru
objectclass                   : {top, person, organizationalPerson, user}
lastlogontimestamp            : 4/8/2023 7:03:41 PM
name                          : jaki
objectsid                     : S-1-5-21-4078382237-1492182817-2568127209-7682
samaccountname                : jaki
codepage                      : 0
samaccounttype                : USER_OBJECT
accountexpires                : NEVER
countrycode                   : 0
whenchanged                   : 8/25/2023 10:04:43 PM
instancetype                  : 4
usncreated                    : 69311
objectguid                    : edb118e8-3995-45d9-89f1-bf978e4e7fa4
lastlogoff                    : 1/1/1601 2:00:00 AM
objectcategory                : CN=Person,CN=Schema,CN=Configuration,DC=domain,DC=ru
dscorepropagationdata         : {8/25/2023 10:04:43 PM, 1/1/1601 12:00:00 AM}
lastlogon                     : 4/9/2023 1:21:20 PM
badpwdcount                   : 0
cn                            : jaki
useraccountcontrol            : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
whencreated                   : 4/8/2023 9:07:56 AM
primarygroupid                : 513
pwdlastset                    : 4/8/2023 12:07:56 PM
msds-supportedencryptiontypes : 0
usnchanged                    : 118956

```

So we found out that jaki can add himself to the ServiceMgmt group, although as you can see BloodHound did not know this.

![Image](/img/acl_ad_cd/1.png)


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

[back](/blog)