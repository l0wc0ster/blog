[back](/)

### Working with Redis

Very often I come across the database Redis. Recently, it has become especially popular, the advantage of this database is associated with the speed of its work, as well as a simple implementation.
Access to the Redis database is carried out using a password, and there is also no distinction with users or groups. The base is not a relational, the structure is as follows: "Key" - "Value". The default "Redis Key-Value Store" service uses port 6379/TCP.
Consider several interesting cases associated with the Redis database.

Case 1:
«Portable Kanban» is a planning management application, uses Redis database. To access the database, a password is required, which can be obtained in encrypted form from the PortableKanban.cfg file. Taking the basis of [exploit](https://www.exploit-db.com/exploits/49409) decrypt the password:

![Image](/img/reids/1.png)



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