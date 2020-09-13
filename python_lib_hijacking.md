[back](/)

### Privilege Escalation via Python Library Hijacking

Example 1. You have a script such as backup.py in a directory with file write permissions. If the script contains import of libraries, for example «import zipfile», then you can create a reverse or bind shell named «zipfile.py» in the same directory, the Python interpreter will run it before looking at the libraries in the standard directories:

```
python3 -c'import sys; print ("\ n" .join (sys.path)) '

/usr/lib/python35.zip
/usr/lib/python3.5
/usr/lib/python3.5/plat-x86_64-linux-gnu
/usr/lib/python3.5/lib-dynload
/usr/local/lib/python3.5/dist-packages
/usr/lib/python3/dist-packages
```
![Image](/img/python_lib_hijacking/1.png)

Example 2. You have a script such as backup.py in a directory without write permissions. In this, check for the possibility of writing any of the standard directories where the library is located, for example for zipfile.py it can be /usr, /usr/lib, /usr/lib/python3.7/ if the script is written with Python 3.7. The Python interpreter will run zipfile.py before looking at the last destination directory.

Example 3. You have a script such as backup.py. For a low-privilege user, there is an entry in /etc/sudoers for example "(ALL) SETENV: /opt/scripts/backup.py". In this case, you can create a script zipfile.py with a reverse or bind shell in any directory with the ability to write: /tmp, /dev/shm, etc., and then run it as follows: "sudo PYTHONPATH =/tmp /opt/scripts/backup.py". In this case, the python interpreter will include in the environment the directory specified in "PYTHONPATH" and run the script before looking at the libraries in the standard directories.

![Image](/img/python_lib_hijacking/2.png)

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