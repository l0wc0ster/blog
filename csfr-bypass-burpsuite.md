When you need to bypass csrf token in web application by burp, just remember few rules:

* choose the request which with 100% probability will return a new token
* ask a question, what would you like to see in the request before sending it?
* control your hosts in proxy scopes and use exact entry into the rule
* make the regular expression of the macro and rule settings the most accurate
* and most importantly - describe the logic of the macros "before sending the main request."
For example, if you make brute-force attack on login page, describe the previous get request to return token from web server, do not describe the next step, you can do it in intruder later =) #antiscrf #burp #burpsuite #csrf
