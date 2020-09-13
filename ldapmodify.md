[back](/)

### Privilege escalation kill chain: ldapmodify -> docker gid -> root

Everyone knows well that a user in a docker group (gid = docker, groups = docker) will be able to increase privileges to root. There is a great way from GTFOBins "docker run -v /: / mnt --rm -it ubuntu chroot / mnt sh".

Recently I found myself in an interesting situation when a user had full access to the LDAP, had the ability to add, remove or change any attributes. But the current user was not in the docker group.

To add to groups and then increase privileges, I used the ldapmodify utility, having previously created an "import.ldif" file. By assigning the gidNumber of the docker group and the uidNumber of the existing local user, as well as adding the sshPublicKey attribute with a public key, you can connect as a domain user and increase the privileges to root.

Of course, it was possible to assign gidNumber = 0, but unfortunately, "PermitRootLogin yes" was not set in sshd_config for security reasons and the key entry option did not work.

![Image](/img/ldapmodify/1.png)
![Image](/img/ldapmodify/2.png)
![Image](/img/ldapmodify/3.png)

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
