### Unprotected admin functionality

Access to /robots.txt. You'l see the /administrator-panel endpoint. Access and delete the user "Carlos"

---
### Unprotected admin functionality with unpredictable URL

Go to the target, see the files, and you'll see the endpoint `/admin-9026a1`. Then delete the "Carlos" user.

---
### User role controlled by request parameter

Access the /admin panel, and you'll see there is a cookie `Admin=false`. Intercept all requests and change it to `Admin=true` and delete Carlos account.

---
### User role can be modified in user profile

Update your email with a new one, and add the parameter "roleid":2 to the JSON. Then access the `/admin` endpoint, and delete the Carlos user. 

---
### User ID controlled by request parameter

Once you log in, you'll see your API key and the endpoint is `/my-account=id=wiener` change that for "carlos", and submit the API key as the solution.

---
### User ID controlled by request parameter, with unpredictable user IDs

In this case, you have a UUID assigned to your username, and the /my-account can be accessed with the id=UUID. 
Go to a blog post published by Carlos, and see it's disclosing his UUID. Change it in the endpoint, and you can get his API key value. 

---
### User ID controlled by request parameter with data leakage in redirect

Change your id to `carlos` in the URL. You'll be redirected to the login page. But in the redirect response, you can find your API key. Submit that as the solution. 

---
### User ID controlled by request parameter with password disclosure

Change your id to `administrator` and see the password in the response. Log in, and delete the account.

---
### Insecure direct object references

Send a message on the Live chat. Download the transcript, and see it's a GET to `/download-transcript/2.txt` change it to "1.txt" and you'll find the password.

---
### URL-based access control can be circumvented

Try accessing to `/admin` and you'll get an access denied. 

Make a request to / and add the header `X-Original-URL: /admin` and you can see the admin panel. 

So, now create a GET to `/?username=carlos` and add the `X-Original-URL: /admin/delete`. Solved.

---
### Method-based access control can be circumvented

As the admin, upgrade the user "Carlos". Send that request to the repeater, and send it again, but with the Wiener session cookie. 
You'll get "Unauthorized". Switch from POST to POSTX, and you'll get missing parameter error. 

Transform that into a GET request, and it's working. Change the username from carlos to wiener. 

---
### Multi-step process with no access control on one step

Log in as admin, upgrade Carlos. Send that request to repeater. 
Log in as Wiener, get the session cookie, put it on the upgrade request, and add the parameter "confirmed=true". That's it.

---
### Referer-based access control

Log in as admin, upgrade Carlos. Go to the same endpoint as Wiener, you get unauth due to the lack of the "Referer" header. 
Use Wiener cookie on the admin request, and change the username to Wiener. That's it.

