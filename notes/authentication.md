
# Lab 1

## Description

This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:

- [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.


## Solution

Use burp intruder with the the login POST request. Enumerating username first yields `alerts` as username. Rerun the bruteforce with the password list which returns `biteme`.  Logging in as user `alerts` solve the lab.

Hydra could be used here for faster bruteforcing, but the list is short enough for burp community to be enough.

# Lab 2

## Description

This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code. To solve the lab, access Carlos's account page.

- Your credentials: `wiener:peter`
- Victim's credentials `carlos:montoya`


## Solution

Login with the attacker credentials shows the login process:

- /login : asking for username:password
- /login2 : asking for security code

However, when completing the first login page (using  credentials), the account login is already validated  - the server returns a valid session cookie that can be used to access `/my-account` page directly without needing the verification code.


# Lab 3

## Description

This lab's password reset functionality is vulnerable. To solve the lab, reset Carlos's password then log in and access his "My account" page.

- Your credentials: `wiener:peter`
- Victim's username: `carlos`

## Solution

Completing the password reset process with `wiener` user with following steps:

- /forgot-password
- enter the username
- receive reset password link
- follow the link
- change the password

Looking at the request changing the password shows:

```
POST /forgot-password?temp-forgot-password-token=ayasj1i6wgd3cdamrsmdz8d1ruw7cwls HTTP/2
Host: 0a7900fd044cc36180635376002f00a5.web-security-academy.net
[...]
temp-forgot-password-token=ayasj1i6wgd3cdamrsmdz8d1ruw7cwls&username=wiener&new-password-1=test&new-password-2=test
```

The username is send with the reset request. Changing it to value `carlos`:

```
POST /forgot-password?temp-forgot-password-token=ayasj1i6wgd3cdamrsmdz8d1ruw7cwls HTTP/2
Host: 0a7900fd044cc36180635376002f00a5.web-security-academy.net
[...]
temp-forgot-password-token=ayasj1i6wgd3cdamrsmdz8d1ruw7cwls&username=carlos&new-password-1=test&new-password-2=test
```

changes the password to `test` and enables the attacker to login as `carlos` user and solve the lab.

