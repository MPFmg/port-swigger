### JWT authentication bypass via unverified signature

Admin interface only available if logged in as an administrator

Send the GET to /my-account to the repeater, inspect the payload (2nd part) of the JWT and modify the "sub" from wiener to administrator
Now send a GET to `/admin/delete?username=carlos`

---
### JWT authentication bypass via flawed signature verification

Modify the JWT header, change the alg to "none" and the payload, change the sub to "administrator". Remove all the signature of the JWT, but leave the `.` 
GET to `/admin/delete?username=carlos`

---
### JWT authentication bypass via weak signing key

Copy the JWT, and use this command:
`hashcat -a 0 -m 16500 <YOUR JWT> /path/to/jwt.secrets.list`

After it ends, append the flag `--show`
Encode the key in base64 on Burp's decoder. 

On JWT editor, create a new symmetric key, generate, and replace the "k" value with your base64. On the repeater, in the new JWT tab, modify the "sub" to administrator. Sign it with your key, and delete Carlos account.

---
### JWT authentication bypass via jwk header injection

Same as previous, go to the JWT Editor Keys tab, create RSA key. 
Replace the value "sub" for "administrator" on the JWT. Click on "Attack" > Embedded JWK. 

You should be able to delete Carlos account.

---
### JWT authentication bypass via jku header injection

Log in, send the /my-account to the repeater. 
Go to the JWT Editor Keys, create a new RSA. 

In the exploit server, paste this as body:
```text
{
    "keys": [

    ]
}
```

Now right click on your freshly created RSA, and click on "Copy Public Key as JWK". Paste it inside the "[]" of the body in the exploit server. **Store it.**

In the request on the repeater, set it to /admin, and modify the JWT. Change the "kid" for the one you pasted in the exploit server.
Add a parameter `"jku":"<EXPLOIT-SERVER>/exploit"`
Modify the "sub" to "administrator".

Click on sign. Select the RSA key. Select "Don't modify the header". Send the request, and delete Carlos' account

---
### JWT authentication bypass via kid header path traversal

Send the /my-account request to the repeater. 

On the JWT editor tab, create a Symmetric Key, and replace the "k" value with `"AA=="` 
Now on the repeater, go to JSON Web Token label, and replace the kid with `../../../../../../../dev/null` and the "sub" to "administrator". 

Delete the carlos account. 