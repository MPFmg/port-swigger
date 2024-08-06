### Information disclosure in error messages

Replace the "productId=1" for "productId=example"

Solution: `Apache Struts 2 2.3.31`

---
### Information disclosure on debug page

Go to the target, site map, and look for the "cgi-bin" directory. Inside you'll find the phpinfo.php file. Send the request to the repeater, and look in the reponses for the SECRET_KEY.

Solution: `7ugsp1uulbs2mv8qwg88f3p73h81oi42`

---
### Source code disclosure via backup files

Go to /robots.txt > /backup
There is a file to connect to a posgresql ddbb, along with a password. That's the key.

Solution: `6gmakr935becwdque2wd6uxq5tcyju6k`

---
### Authentication bypass via information disclosure

Send a GET to /admin. And then a TRACE /admin. See there is a new header.
In Proxy > Options > Match and Replace > Add:
Request Header
Condition: Blank
Replace: X-Custom-IP-Authorization: 127.0.0.1

delete the username Carlos

---
### Information disclosure in version control history

Access the lag, and go to /.git
`wget -r <YOUR LAB>`

Open git-cola, and look go to "Commit" > "Undo last commit". You'll find the admin password:

Solution: `ywhn7kld1sng8z32ja67`