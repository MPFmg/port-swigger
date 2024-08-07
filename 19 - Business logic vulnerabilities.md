### Excessive trust in client-side controls

When you add the leather jacket, it sends a body like this:
`productId=1&redir=PRODUCT&quantity=1&price=YOUR-PRICE`

Change the price to 1, and buy.

---
### High-level logic vulnerability

Add one jacket, and some other item in negative quantity, so the amount is available for you to purchase.

---
### Inconsistent security controls

- Open the lab then go to the "Target" > "Site map" tab in Burp. Right-click on the lab domain and select "Engagement tools" > "Discover content" to open the content discovery tool.
- Click "Session is not running" to start the content discovery. After a short while, look at the "Site map" tab in the dialog. Notice that it discovered the path `/admin`.

After this, register following the normal flow. And then in "my-account" change your email to something@dontwannacry.com and you can access the /admin panel. 

---
### Flawed enforcement of business rules

You have a discount code when you log in, and another at the bottom when you subscribe to the newsletter. 

Go to the cart, and apply both, switching between them, until you have a value of 0 to pay. 

---
### Low-level logic flaw

If you send a lot of leather jackets, you'll see eventually it will show a big negative value on the cart. Send "323" requests of the jacket with 99 on quantity, then add 47 leather jackets and 16 Hologram Stand in, and buy it. 

---
### Inconsistent handling of exceptional input

Go to the target > Engagement tools > Discover conent

You'll find a /admin but a message says: `Admin interface only available if logged in as a DontWannaCry user`

So, register an account, with a very long string as email, and see it's capping the value to 255.

Your email payload will be `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA@dontwannacry.com` which are 255 characters exactly until the "m" and then append the rest of your attacker email: exploit-0a27009b0419887f83687caa01bb00ba.exploit-server.net

Log in, access /admin and delete Carlos user

---
### Weak isolation on dual-use endpoint

Log in, change your password, and check that if you remove the "current-password" parameter from the request, can change the password without providing the current one.

Set the username for the administrator, change the password, log in and delete carlos account. 

---
### Insufficient workflow validation

log in, buy something you actually can.
When you place an order, the POST to /cart/checkout redirects you to the confirmation page. Send the GET to that confirmation endpoint to repeater `/cart/order-confirmation?order-confirmation=true`

Ad the jacket, resend the order confirmation. 

---
### Authentication bypass via flawed state machine

Login, and you need to select your role before going to the home page. With the discovery tool, you'll find out there is a `/admin` endpoint. 

Intercept ON, then log in, and drop the role-selector action. Then navigate to the home page and delete the Carlos account.

---
### Infinite money logic flaw

1. With Burp running, log in and sign up for the newsletter to obtain a coupon code, `SIGNUP30`. Notice that you can buy $10 gift cards and redeem them from the "My account" page.
2. Add a gift card to your basket and proceed to the checkout. Apply the coupon code to get a 30% discount. Complete the order and copy the gift card code to your clipboard.
3. Go to your account page and redeem the gift card. Observe that this entire process has added $3 to your store credit. Now you need to try and automate this process.
4. Study the proxy history and notice that you redeem your gift card by supplying the code in the `gift-card` parameter of the `POST /gift-card` request.
5. Go to "Project options" > "Sessions". In the "Session handling rules" panel, click "Add". The "Session handling rule editor" dialog opens.
6. In the dialog, go to the "Scope" tab. Under "URL Scope", select "Include all URLs".
7. Go back to the "Details" tab. Under "Rule actions", click "Add" > "Run a macro". Under "Select macro", click "Add" again to open the Macro Recorder.
8. Select the following sequence of requests:
    
    `POST /cart POST /cart/coupon POST /cart/checkout GET /cart/order-confirmation?order-confirmed=true POST /gift-card`
    
    Then, click "OK". The Macro Editor opens.
    
9. In the list of requests, select `GET /cart/order-confirmation?order-confirmed=true`. Click "Configure item". In the dialog that opens, click "Add" to create a custom parameter. Name the parameter `gift-card` and highlight the gift card code at the bottom of the response. Click "OK" twice to go back to the Macro Editor.
10. Select the `POST /gift-card` request and click "Configure item" again. In the "Parameter handling" section, use the drop-down menus to specify that the `gift-card` parameter should be derived from the prior response (response 4). Click "OK".
11. In the Macro Editor, click "Test macro". Look at the response to `GET /cart/order-confirmation?order-confirmation=true` and note the gift card code that was generated. Look at the `POST /gift-card` request. Make sure that the `gift-card` parameter matches and confirm that it received a `302` response. Keep clicking "OK" until you get back to the main Burp window.
12. Send the `GET /my-account` request to Burp Intruder. Use the "Sniper" attack type.
13. On the "Payloads" tab, select the payload type "Null payloads". Under "Payload settings", choose to generate `412` payloads.
14. Go to the "Resource pool" tab and add the attack to a resource pool with the "Maximum concurrent requests" set to `1`. Start the attack.
15. When the attack finishes, you will have enough store credit to buy the jacket and solve the lab.

---
### Authentication bypass via encryption oracle

Watch this video https://www.youtube.com/watch?v=62spVp-GVPI