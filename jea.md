[back](/blog)

### Arbitary file read using Powershell JEA (Just Enough Administration)

Just Enough Administration (JEA) is a technology that allows you to delegate the administration of various windows components, first appeared in Powershell 5.0. JEA allows you to differentiate user rights to use commands, create functions for specific users of the system.
In order to better understand the purpose of this technology, imagine that you need to delegate control of the DNS server to a specific domain user, without adding a user to the "Domain Admins" group, this will be very difficult to do, especially if the DC roles are within the same server. However, using JEA technology, it is possible to give the user minimal rights to administer the DNS server, for example, to restart the service or restore the cache.

In this article, I'll show you how to create a JEA mount point, as well as an interesting way to read arbitrary files from the admin directory.

Let's create a JEA connection point:

First of all, let's create a .pssc configuration file:

```
New-PSSessionConfigurationFile -Path 'C:\Program Files\WindowsPowerShell\test.pssc'
```

![Image](/img/jea/1.png)

The main parameters that need to be edited in this file:

**SessionType - RestrictedRemoteServer**, will allow the following commands:

![Image](/img/jea/2.png)

**TranscriptDirectory - C:\logdir**, directory where session log files will be created

![Image](/img/jea/3.png)

**RunAsVirtualAccount - $true**, start a session from under the virtual administrator account, this virtualization attribute allows you to run the commands specified in the configuration on behalf of the virtual administrator

**RoleDefinitions - @ {'hostname\username' = @ {'RoleCapabilities' = 'group_name'}}**, description of the user and group with the ability to remotely connect to the session

Check the validity of the configuration file

```
Test-PSSessionConfigurationFile -Path 'C:\Program Files\WindowsPowerShell\test.pssc' -verbose
```

![Image](/img/jea/4.png)

Create a .psrc role description file, the file name name must be identical to the value of the 'RoleCapabilities' attribute

```
New-Item -Path 'C:\Program Files\WindowsPowerShell\Modules\JEA\RoleCapabilities' -ItemType Directory
New-PSRoleCapabilityFile -Path 'C:\Program Files\WindowsPowerShell\Modules\JEA\RoleCapabilities\test.psrc'
```

This file describes all the functions and commands that will be available to the user within this session.

![Image](/img/jea/5.png)

In the current example, the "Check-File" function has been created in the file with the role description, which allows you to read the contents of any files in the "D:\*" or "C:\ProgramData*" directory

Now let's register the new configuration:

```
Register-PSSessionConfiguration -Name testadmin -Path 'C:\Program Files\WindowsPowerShell\test.pssc'

```

![Image](/img/jea/6.png)

Next, restart WinRM and check that a new JEA connection point has appeared:

```
Restart-Service WinRM
Get-PSSessionConfiguration | ft name
```

![Image](/img/jea/7.png)

Done, let's try to log into the WinRM session using the new JEA configuration:

![Image](/img/jea/8.png)

Everything works, and we have a strictly limited set of commands, beyond which it is impossible to go, but you can try to use the created function "Check-File" to read arbitrary files with administrator privileges =)

To do this, we need a user with link creation rights in the C:\ProgramData directory:

```
New-Item -ItemType Junction -Path 'C:\ProgramData\root' -Target 'C:\Users\Administrator'
```

![Image](/img/jea/9.png)

Now let's try to read the file that is in the administrator directory:

![Image](/img/jea/10.png)

Thus, we got full read access to the files in the system. Despite the fact that JEA provides a wonderful opportunity for delegation and differentiation of administrative rights, you need to be careful when creating your own functions =)

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

