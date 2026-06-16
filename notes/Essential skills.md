
# Lab 1

## Description

This lab contains a vulnerability that enables you to read arbitrary files from the server. To solve the lab, retrieve the contents of `/etc/passwd` within 10 minutes.

Due to the tight time limit, we recommend using Burp Scanner to help you. You can obviously scan the entire site to identify the vulnerability, but this might not leave you enough time to solve the lab. Instead, use your intuition to identify endpoints that are likely to be vulnerable, then try running a [targeted scan on a specific request](https://portswigger.net/web-security/essential-skills/using-burp-scanner-during-manual-testing#scanning-a-specific-request). Once Burp Scanner has identified an attack vector, you can use your own expertise to find a way to exploit it.

## Solution

