### File path traversal, simple case

There is a request pointing to /image?filename=39.jpg. You can change the filename for the desired one, in this case `../../../../etc/passwd`

---
### File path traversal, traversal sequences blocked with absolute path bypass

Same as previous, but in this case you need to indicate absolute path filename=/etc/passwd

---
### File path traversal, traversal sequences stripped non-recursively

The input is not properly sanitized, allowing you to do the following:
`....//....//....//....//etc/passwd` 
Its not sanitizing recursively.

---
### File path traversal, traversal sequences stripped with superfluous URL-decode

Get a request calling the `/image?filename=` and replace the filename for: `..%252f..%252f..%252fetc/passwd`. This is encoding the `%` symbol as `%25` to read the /etc/passwd file.

---

### File path traversal, validation of start of path

Here it's validating the start of the path to be match `/var/www/images/` so you can append `../../../etc/passwd` and access. 

---
### File path traversal, validation of file extension with null byte bypass

Here you can bypass the restriction by using a null byte "%00" `../../../etc/passwd%00.png` 

Null byte is a bypass technique for sending data that would be filtered otherwise. It relies on injecting the null byte characters (`%00`, `\x00`) in the supplied data. Its role is to terminate a string.

