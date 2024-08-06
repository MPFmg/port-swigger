# port-swigger

https://medium.com/@edgenask/burp-suite-certified-practitioner-bscp-review-tips-and-comparison-with-ewpt-7048d36314d2
https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study/blob/main/README.md

## [[01 - SQL Injection]] DONE
- [x] SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
- [x] SQL injection vulnerability allowing login bypass
- [x] SQL injection attack, querying the database type and version on Oracle
- [x] SQL injection attack, querying the database type and version on MySQL and Microsoft
- [x] SQL injection attack, listing the database contents on non-Oracle databases
- [x] SQL injection attack, listing the database contents on Oracle
- [x] SQL injection UNION attack, determining the number of columns returned by the query
- [x] SQL injection UNION attack, finding a column containing text
- [x] SQL injection UNION attack, retrieving data from other tables
- [x] SQL injection UNION attack, retrieving multiple values in a single column
- [x] Blind SQL injection with conditional responses
- [x] Blind SQL injection with conditional errors
- [x] Visible error-based SQL injection
- [x] Blind SQL injection with time delays
- [x] Blind SQL injection with time delays and information retrieval
- [x] Blind SQL injection with out-of-band interaction
- [x] Blind SQL injection with out-of-band data exfiltration
- [x] SQL injection with filter bypass via XML encoding
##  [[02 - Cross-site Scripting (XSS)]] DONE
- [x] Reflected XSS into HTML context with nothing encoded
- [x] Stored XSS into HTML context with nothing encoded
- [x] DOM XSS in `document.write` sink using source `location.search`
- [x] DOM XSS in `innerHTML` sink using source `location.search`
- [x] DOM XSS in jQuery anchor `href` attribute sink using `location.search` source
- [x] DOM XSS in jQuery selector sink using a hashchange event
- [x] Reflected XSS into attribute with angle brackets HTML encoded
- [x] Stored XSS into anchor href attribute with double quotes HTML encoded
- [x] Reflected XSS into a JavaScript string with angle brackets HTML encoded
- [x] DOM XSS in `document.write` sink using source `location.search` inside a select element
- [x] DOM XSS in AngularJS expression with angle brackets and double quotes HTML encoded
- [x] Reflected DOM XSS
- [x] Stored DOM XSS
- [x] Reflected XSS into HTML context with most tags and attributes blocked
- [x] Reflected XSS into HTML context with all tags blocked except custom ones
- [x] Reflected XSS with some SVG markup allowed
- [x] Reflected XSS in canonical link tag
- [x] Reflected XSS into a JavaScript string with single quote and backslash escaped
- [x] Reflected XSS into a JavaScript string with angle brackets and double quotes HTML encoded and single quotes escaped
- [x] Stored XSS into `onclick` event with angle brackets and double quotes HTML encoded single quotes and backslash escaped
- [x] Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
- [x] Exploiting cross-site scripting to steal cookies
- [x] Exploiting cross-site scripting to capture passwords (\*\*)
- [x] Exploiting XSS to perform CSRF

## [[03 - CSRF]] DONE
- [x] CSRF vulnerability with no defenses
- [x] CSRF where token validation depends on request method 
- [x] CSRF where token validation depends on token being present
- [x] CSRF where token is not tied to user session
- [x] CSRF where token is tied to non-session cookie
- [x] CSRF where token is duplicated in cookie
- [x] SameSite Lax bypass via method override
- [x] SameSite Strict bypass via client-side redirect
- [x] SameSite Strict bypass via sibling domain
- [x] SameSite Lax bypass via cookie refresh
- [x] CSRF where Referer validation depends on header being present
- [x] CSRF with broken Referer validation

## [[04 - Clickjacking]] DONE
- [x] Basic clickjacking with CSRF token protection
- [x] Clickjacking with form input data prefilled from a URL parameter
- [x] Clickjacking with a frame buster script
- [x] Exploiting clickjacking vulnerability to trigger DOM-based XSS
- [x] Multistep clickjacking
 
## [[05 - DOM-based vulnerabilities]] DONE
- [x] DOM XSS using web messages
- [x] DOM XSS using web messages and a JavaScript URL
- [x] DOM XSS using web messages and `JSON.parse`
- [x] DOM-based open redirection
- [x] DOM-based cookie manipulation
## [[06 - Cross-origin resource sharing (CORS)]] DONE
- [x] CORS vulnerability with basic origin reflection
- [x] CORS vulnerability with trusted null origin
- [x] CORS vulnerability with trusted insecure protocols

## [[07 - XML external entity (XXE) injection]] DONE
- [x] Exploiting XXE using external entities to retrieve files
- [x] Exploiting XXE to perform SSRF attacks
- [x] Blind XXS with out-of-band interaction
- [x] Blind XXE with out-of-band interaction via XML parameter entities
- [x] Exploiting blind XXE to exfiltrate data using a malicious external DTD
- [x] Exploiting blind XXE to retrieve data via error messages
- [x] Exploiting XInclude to retrieve files
- [x] Exploiting XXE via image file upload

## [[08 - Server-side request forgery (SSRF)]] DONE
- [x] Basic SSRF against the local server
- [x] Basic SSRF against another back-end system
- [x] Blind SSRF with out-of-band detection
- [x] SSRF with blacklist-based input filter
- [x] SSRF with filter bypass via open redirection vulnerability

## [[09 - HTTP Request Smuggling]] DONE
- [x] HTTP request smuggling, confirming a CL.TE vulnerability via differential responses
- [x] HTTP request smuggling, confirming a TE.CL vulnerability via differential responses
- [x] Exploiting HTTP request smuggling to bypass front-end security controls, CL.TE vulnerability
- [x] Exploiting HTTP request smuggling to bypass front-end security controls, TE.CL vulnerability
- [x] Exploiting HTTP request smuggling to reveal front-end request rewriting
- [x] Exploiting HTTP request smuggling to capture other users' requests
- [x] Exploiting HTTP request smuggling to deliver reflected XSS 
- [x] Response queue poisoning via H2.TE request smuggling
- [x] H2.CL request smuggling
- [x] HTTP/2 request smuggling via CRLF injection
- [x] HTTP/2 request splitting via CRLF injection
- [ ] CL.0 request smuggling
- [x] HTTP request smuggling, basic CL.TE vulnerability
- [x] HTTP request smuggling, basic TE.CL vulnerability
- [x] HTTP request smuggling, obfuscating the TE header

## [[10 - OS command injection]] DONE
- [x] OS command injection, simple case
- [x] Blind OS command injection with time delays
- [x] Blind OS command injection with output redirection
- [x] Blind OS command injection with out-of-band interaction
- [x] Blind OS command injection with out-of-band data exfiltration

## [[11 - Server-side template injection]] DONE
- [x] Basic server-side template injection
- [x] Basic server-side template injection (code context)
- [x] Server-side template injection using documentation
- [x] Server-side template injection in an unknown language with a documented exploit
- [x] Server-side template injection with information disclosure via user-supplied objects

## [[12 - Path Traversal]] DONE
- [x] File path traversal, simple case
- [x] File path traversal, traversal sequences blocked with absolute path bypass
- [x] File path traversal, traversal sequences stripped non-recursively
- [x] File path traversal, traversal sequences stripped with superfluous URL-decode
- [x] File path traversal, validation of start of path
- [x] File path traversal, validation of file extension with null byte bypass

## [[13 - Access control vulnerabilities]] DONE
- [x] Unprotected admin functionality
- [x] Unprotected admin functionality with unpredictable URL
- [x] User role controlled by request parameter
- [x] User role can be modified in user profile
- [x] User ID controlled by request parameter
- [x] User ID controlled by request parameter, with unpredictable user ID's
- [x] User ID controlled by request parameter with data leakage in redirect
- [x] User ID controlled by request parameter with password disclosure
- [x] Insecure direct object references
- [x] URL-based access control can be circumvented
- [x] Method-based access control can be circumvented
- [x] Multi-step process with no access control on one step
- [x] Referer-based access control

## [[14 - Authentication]] DONE
- [x] Username enumeration via different responses
- [x] 2FA simple bypass
- [x] Password reset broken logic
- [x] Username enumeration via subtly different responses
- [x] Username enumeration via response timing
- [x] Broken brute-force protection, IP block
- [x] Username enumeration via account lock
- [x] 2FA broken logic
- [x] Brute-forcing a stay-logged-in cookie
- [x] Offline password cracking
- [x] Password reset poisoning via middleware
- [x] Password brute-force via password change

## [[15 - WebSockets]] DONE
- [x] Manipulating WebSocket messages to exploit vulnerabilities
- [x] Cross-site WebSocket hijacking
- [x] Manipulating the WebSocket handshake to exploit vulnerabilities

## [[16 - Web cache poisoning]] DONE
- [x] Web cache poisoning with an unkeyed header
- [x] Web cache poisoning with an unkeyed cookie
- [x] Web cache poisoning with multiple headers
- [x] Targeted web cache poisoning using an unknown header
- [x] Web cache poisoning via an unkeyed query string
- [x] Web cache poisoning via an unkeyed query parameter
- [x] Parameter cloaking
- [x] Web cache poisoning via a fat GET request
- [x] URL normalization

## [[17 - Insecure Deserialization]] DONE
- [x] Modifying serialized objects
- [x] Modifying serialized data types
- [x] Using application functionality to exploit insecure deserialization
- [x] Arbitrary object injection in PHP
- [x] Exploiting Java deserialization with Apache Commons
- [x] Exploiting PHP deserialization with a pre-built gadget chain
- [x] Exploiting Ruby deserialization using a documented gadget chain

## [[18 - Information disclosure]] DONE
- [x] Information disclosure in error messages
- [x] Information disclosure on debug page
- [x] Source code disclosure via backup files
- [x] Authentication bypass via information disclosure
- [x] Information disclosure in version control history

## [[19 - Business logic vulnerabilities]] DONE
- [x] Excessive trust in client-side controls
- [x] High-level logic vulnerability
- [x] Inconsistent security controls
- [x] Flawed enforcement of business rules
- [x] Low-level logic flaw
- [x] Inconsistent handling of exceptional input
- [x] Weak isolation on dual-use endpoint
- [x] Insufficient workflow validation
- [x] Authentication bypass via flawed state machine
- [x] Infinite money logic flaw
- [x] Authentication bypass via encryption oracle
 
## [[20 - HTTP Host header]] DONE
- [x] Basic password reset poisoning
- [x] Host header authentication bypass
- [x] Web cache poisoning via ambiguous requests
- [x] Routing-based SSRF
- [x] SSRF via flawed request parsing
- [x] Host validation bypass via connection state attack
## [[21 - OAuth Authentication]] DONE
- [x] Authentication bypass via OAuth implicit flow
- [x] SSRF via OpenID dynamic client registration
- [x] Forced OAuth profile linking
- [x] OAuth account hijacking via redirect_uri
- [x] Stealing OAuth access tokens via an open redirect

## [[22 - File Upload]] DONE
- [x] Remote code execution via web shell upload
- [x] Web shell upload via Content-Type restriction bypass
- [x] Web shell upload via path traversal
- [x] Web shell upload via extension blacklist bypass 
- [x] Web shell upload via obfuscated file extension
- [x] Remote code execution via polyglot web shell upload

## [[23 - JWT]] DONE
- [x] JWT authentication bypass via unverified signature
- [x] JWT authentication bypass via flawed signature verification
- [x] JWT authentication bypass via weak signing key
- [x] JWT authentication bypass via jwk header injection
- [x] JWT authentication bypass via jku header injection
- [x] JWT authentication bypass via kid header path traversal

## [[24 - Essential skills]]DONE
- [x] Discovering vulnerabilities quickly with targeted scanning
- [x] Scanning non-standard data structures

## [[25 - Prototype pollution]] DONE
- [x] Client-side prototype pollution via browser APIs
- [x] DOM XSS via client-side prototype pollution
- [x] DOM XSS via an alternative prototype pollution vector
- [x] Client-side prototype pollution via flawed sanitization
- [x] Client-side prototype pollution in third-party libraries
- [x] Privilege escalation via server-side prototype pollution
- [x] Detecting server-side prototype pollution without polluted property reflection
- [x] Bypassing flawed input filters for server-side prototype pollution
- [x] Remote code execution via server-side prototype pollution

## [[26 - GraphQL API vulnerabilities]] DONE
- [x] Accessing private GraphQL posts
- [x] Accidental exposure of private GraphQL fields
- [x] Finding a hidden GraphQL endpoint
- [x] Bypassing GraphQL brute force protections
- [x] Performing CSRF exploits over GraphQL

## [[27 - Race Conditions]] DONE
- [x] Limit overrun race conditions
- [x] Bypassing rate limits via race conditions
- [x] Multi-endpoint race conditions
- [x] Single-endpoint race conditions
- [x] Exploiting time-sensitive vulnerabilities
## [[28 - NoSQL injection]] 1
- [x] Detecting NoSQL injection
- [x] Exploiting NoSQL operator injection to bypass authentication
- [x] Exploiting NoSQL injection to extract data
- [ ] Exploiting NoSQL operator injection to extract unknown fields
## [[29 - API testing]] DONE
- [x] Exploiting an API endpoint using documentation
- [x] Exploiting server-side parameter pollution in a query string
- [x] Finding and exploiting an unused API endpoint
- [x] Exploiting a mass assignment vulnerability
- [x] Exploiting server-side parameter pollution in a REST URL
## [[30 - Web LLM attacks]] DONE
 - [x] Exploiting LLM APIs with excessive agency
 - [x] Exploiting vulnerabilities in LLM APIs
 - [x] Indirect prompt injection

