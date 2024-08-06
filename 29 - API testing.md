### Exploiting an API endpoint using documentation

Log in, change your email. See there is a PATCH request to /api/user/wiener

Send a GET request to /api and see it's disclosing API documentation on how to DELETE a user. Use that info to delete Carlos account

---
### Exploiting server-side parameter pollution in a query string

trigger a password reset for the admin, and notice it's related to a /js/forgotPassword.js file. 

Send the POST to /forgot-password, get a 200, send it to an nonexistent username, and get an error. 

If you try to append another parametre, like `username=administrator%26x=y`, it says "Parameter is not supported". 

If you send `username=administrator%23` it says "Field not specified"

Now try `username=administrator%26field=x%23` and get "Invalid field"
Send this to intruder, and add a payload position in the "x". 
Use the built-in "Server-side variable names". And you'll see there is "email".

By looking at the js file, there is a "reset_token" parameter, so change the field to "reset_token" and send it. You'll receive a reset token

Open in browser the endpoint `/forgot-password?reset_roken=<YOUR_TOKEN>`

Log in, delete carlos account.

---
### Finding and exploiting an unused API endpoint

For each product you visit, there is a request to /api/products/1/price

Ok, here the thing is. Send an OPTIONS to that, then a PATCH. It'll ask for the content-type "application/json". Add it.

Then add a body "{}" and then add a price `{"price":0}`

Then buy the jacket.

---
### Exploiting a mass assignment vulnerability

When you try to buy the jacket, it sends a GET and a POST to /api/checkout.

You can use the parameter "chosen_discount" from the response, and ad it to your JSON and add it to your request with a percentage of 100

```json
`{ "chosen_discount":{ "percentage":0 }, "chosen_products":[ { "product_id":"1", "quantity":1 } ] }`
```