[back](/)

### «JWKS Spoofing» and «Open Redirect» friends forever

JWKS is a set of public keys for validating a JWT token signed with the RSA encryption algorithm.

![Image](/img/jwks/3.png)

Imagine a case, in front of you is a site with open user registration, as a result, the user is issued a JWT token with the RS256 signature algorithm.

![Image](/img/jwks/1.png)

The JWT token header contains the **jku** parameter with a pointer to the JWKS for validation.

![Image](/img/jwks/2.png)

At the same time, during the reconnaissance of the site, the **Open Redirect** vulnerability was discovered.

![Image](/img/jwks/4.png)

What can be done to elevate privileges on a web resource with such information? You can get the privileges of any known user, including administrator.
The first thing to do is to change the values of the JWT token headers: **HEADER** and **PAYLOAD**, [jwt_tool](https://github.com/ticarpi/jwt_tool) is suitable for this task
We use the found **Open Redirect** vulnerability to redirect the request to our own malicious server so that the domain validation check is successful. In the next step, we will change the **PAYLOAD** header and indicate the intended user login: **admin**.

![Image](/img/jwks/5.png)

After changing the headers of the JWT token, its checksum has changed, which means that the signature has changed, the **SIGNATURE** header is no longer valid. Let's check what happened on the site [jwt.io](https://jwt.io).

![Image](/img/jwks/6.png)

Now we need to sign our data in **RS256** format, for this task we will generate keys using the **ssh-keygen** and **openssl** utilities. In addition, we need to generate JWKS, so we can use the following code. You can create your own version, or find a ready-made solution on the Internet.

```
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from jwcrypto.jwk import JWK

private = open('jwtRS256.key', 'rb').read()
public = open('jwtRS256.key.pub', 'rb').read()
private_key = serialization.load_pem_private_key(private, password=None, backend=default_backend())
public_key = serialization.load_pem_public_key(public, backend=default_backend())
private_jwk = JWK()
private_jwk.import_from_pyca(private_key)
public_jwk = JWK()
public_jwk.import_from_pyca(public_key)

pr = private_jwk.export_private().encode('utf-8')
print(pr)
print("\n")
pu = public_jwk.export_public().encode('utf-8')
print(pu)
```

![Image](/img/jwks/7.png)

Using [jwt_tool](https://github.com/ticarpi/jwt_tool), we will sign our updated **HEADER** and **PAYLOAD** header data, and set the generated private key to the command input.

![Image](/img/jwks/8.png)

The received JWT token is used for authorization on the target web resource, at the same time it is necessary to start the web server to send JWKS to the application.

![Image](/img/jwks/9.png)

![Image](/img/jwks/10.png)

As a result of the actions performed, we get access to the web resource with administrative privileges.

![Image](/img/jwks/11.png)

![Image](/img/jwks/12.png)

As you can see, thanks to not cunning manipulations, we have raised our privileges and our mood. Stay safe and remember that even minor vulnerabilities can be critical =)

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